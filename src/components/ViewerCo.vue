<template>
  <div v-if="isLoading" class="loading-overlay">
    <div class="loading-box">
      <div class="spinner"></div>
      <p class="loading-text">Loading {{ loadingFileName }}</p>
      <p class="loading-text">{{ loadingProgress }}%</p>
    </div>
  </div>

  <div ref="containerRef" class="full-screen">
    <bim-grid id="appGrid"></bim-grid>

    <input
      ref="ifcInput"
      type="file"
      accept=".ifc"
      style="display: none"
      @change="convertIFC"
    />

    <div class="control-panels">
      <section
  class="flow-panel"
  :class="{ 'flow-panel--minimized': isElementPanelMinimized }"
  aria-label="Element classification controls"
>
        <div class="flow-panel__header">
          <div>
            <p class="flow-panel__eyebrow">Elementos da central</p>
            <h2>Classificação</h2>
          </div>

          <button
            type="button"
            class="flow-panel__toggle"
            @click="toggleElementPanelMinimized"
          >
            {{ isElementPanelMinimized ? '+' : '−' }}
          </button>
        </div>

        <div class="flow-panel__content">
          <p class="selection-count">Selecionados: {{ selectedCount }}</p>

          <div class="selected-mep-info">
  {{ selectedMepElementInfo }}
</div>

          <p class="connection-note">Tipo de elemento</p>

          <div class="flow-actions flow-actions--secondary">
            <button type="button" @click="defineSelectedElementsAs('pipe')">
              Tubo
            </button>

            <button type="button" @click="defineSelectedElementsAs('normallyOpenValve')">
  Válvula NA
</button>

<button type="button" @click="defineSelectedElementsAs('normallyClosedValve')">
  Válvula NF
</button>

            <button type="button" @click="defineSelectedElementsAs('collector')">
              Coletor
            </button>
          </div>

          <div class="flow-actions flow-actions--secondary">
            <button type="button" @click="defineSelectedElementsAs('booster')">
              Booster
            </button>

            <button type="button" @click="defineSelectedElementsAs('reservoirWithResistance')">
              Reserv. c/ resistência
            </button>

            <button type="button" @click="defineSelectedElementsAs('reservoirWithoutResistance')">
              Reserv. s/ resistência
            </button>
          </div>

          <div class="flow-actions flow-actions--single">
            <button
              type="button"
              class="flow-button--danger"
              @click="deleteSelectedElementDefinitions"
            >
              Apagar definição
            </button>
          </div>

          <p class="connection-note">
            Elementos definidos: {{ countDefinedMepElements() }}
          </p>

          <dl class="flow-stats">
            <div>
              <dt>Tubos</dt>
              <dd>{{ countMepElementsByType('pipe') }}</dd>
            </div>

            <div>
              <dt>Válvulas</dt>
<dd>
  {{
    countMepElementsByType('isolationValve') +
    countMepElementsByType('normallyOpenValve') +
    countMepElementsByType('normallyClosedValve')
  }}
</dd>
            </div>

            <div>
              <dt>Colet.</dt>
              <dd>{{ countMepElementsByType('collector') }}</dd>
            </div>

            <div>
              <dt>Booster</dt>
              <dd>{{ countMepElementsByType('booster') }}</dd>
            </div>

            <div>
              <dt>Res. c/ R.</dt>
              <dd>{{ countMepElementsByType('reservoirWithResistance') }}</dd>
            </div>

            <div>
              <dt>Res. s/ R.</dt>
              <dd>{{ countMepElementsByType('reservoirWithoutResistance') }}</dd>
            </div>
          </dl>
          <div class="flow-section-title">
  Realçar elementos
</div>

<div class="element-highlight-list">
  <div class="element-highlight-item">
    <span>Tubos</span>

    <button type="button" @click="highlightMepElementsByType('pipe')">
      Mostrar
    </button>
  </div>

  <div class="element-highlight-item">
  <span>Válvulas NA</span>

  <button type="button" @click="highlightMepElementsByType('normallyOpenValve')">
    Mostrar
  </button>
</div>

<div class="element-highlight-item">
  <span>Válvulas NF</span>

  <button type="button" @click="highlightMepElementsByType('normallyClosedValve')">
    Mostrar
  </button>
</div>

  <div class="element-highlight-item">
    <span>Coletores</span>

    <button type="button" @click="highlightMepElementsByType('collector')">
      Mostrar
    </button>
  </div>

  <div class="element-highlight-item">
    <span>Boosters</span>

    <button type="button" @click="highlightMepElementsByType('booster')">
      Mostrar
    </button>
  </div>

  <div class="element-highlight-item">
    <span>Reserv. c/ resistência</span>

    <button type="button" @click="highlightMepElementsByType('reservoirWithResistance')">
      Mostrar
    </button>
  </div>

  <div class="element-highlight-item">
    <span>Reserv. s/ resistência</span>

    <button type="button" @click="highlightMepElementsByType('reservoirWithoutResistance')">
      Mostrar
    </button>
  </div>
</div>
        </div>
      </section>

<section
  :class="['flow-panel', isFlowControlsPanelMinimized ? 'flow-panel--minimized' : '']"
  aria-label="Water flow controls"
>
  <div class="flow-panel__header">
    <div>
      <p class="flow-panel__eyebrow">Fluxo de água</p>
      <h2>Circuitos hidráulicos</h2>
    </div>

    <button
      type="button"
      class="flow-panel__toggle"
      @click="toggleFlowControlsPanelMinimized"
    >
      {{ isFlowControlsPanelMinimized ? '+' : '−' }}
    </button>
  </div>

  <div class="flow-panel__content">
    <p class="selection-count">Selecionados: {{ selectedCount }}</p>

    <div class="flow-section-title">
      Circuitos
    </div>

    <div class="flow-actions">
      <button type="button" @click="assignSelectedPipes('supply1')">
        Avanço 1
      </button>

      <button type="button" @click="assignSelectedPipes('supply2')">
        Avanço 2
      </button>

      <button type="button" @click="assignSelectedPipes('supply3')">
        Avanço 3
      </button>
    </div>

    <div class="flow-actions flow-actions--secondary">
      <button type="button" @click="assignSelectedPipes('return1')">
        Retorno 1
      </button>

      <button type="button" @click="assignSelectedPipes('return2')">
        Retorno 2
      </button>

      <button type="button" @click="assignSelectedPipes('return3')">
        Retorno 3
      </button>
    </div>

    <div class="flow-actions flow-actions--secondary">
      <button type="button" @click="clearSelectedManualAssignments">
        Limpar marca selecionada
      </button>

      <button type="button" @click="clearManualAssignments">
        Limpar todas as marcas
      </button>

      <button type="button" @click="reverseSelectedPipesDirection">
        Sincronizar sentido
      </button>
    </div>

    <div class="flow-actions flow-actions--secondary">
      <button type="button" @click="hideSelectedFlowArrows">
        Ocultar setas
      </button>

      <button type="button" @click="showSelectedFlowArrows">
        Mostrar setas
      </button>
    </div>

    <div class="flow-section-title">
      Caminhos
    </div>

    <div class="flow-actions flow-actions--single">
  <button
    type="button"
    :class="[
      'manual-route-mode-button',
      isManualRouteRecording ? 'manual-route-mode-button--active' : ''
    ]"
    @click="toggleManualRouteRecording"
  >
    {{ isManualRouteRecording ? 'Desativar modo manual' : 'Ativar modo manual' }}
  </button>
</div>

<p v-if="isManualRouteRecording" class="manual-route-status">
  Modo manual ativo, selecione os tubos pela ordem do percurso.
</p>

<p class="connection-note">
  Caminho manual: {{ manualRouteNodes.length }} tubo(s)
</p>

<div v-if="!isManualRouteRecording" class="flow-actions flow-actions--secondary">
  <button type="button" @click="setRouteStart">
    Definir início
  </button>

  <button type="button" @click="addRouteWaypoint">
    Passar aqui
  </button>

  <button type="button" @click="setRouteEnd">
    Definir fim
  </button>
</div>

<div v-if="!isManualRouteRecording" class="flow-actions">
  <button type="button" @click="createAutoRoute('supply1')">
    Caminho avanço 1
  </button>

  <button type="button" @click="createAutoRoute('supply2')">
    Caminho avanço 2
  </button>

  <button type="button" @click="createAutoRoute('supply3')">
    Caminho avanço 3
  </button>
</div>

<div v-if="!isManualRouteRecording" class="flow-actions flow-actions--secondary">
  <button type="button" @click="createAutoRoute('return1')">
    Caminho retorno 1
  </button>

  <button type="button" @click="createAutoRoute('return2')">
    Caminho retorno 2
  </button>

  <button type="button" @click="createAutoRoute('return3')">
    Caminho retorno 3
  </button>
</div>

<div
  v-if="isManualRouteRecording && manualRouteNodes.length >= 2"
  class="flow-section-title"
>
  Selecionar tipo do caminho manual
</div>

<div
  v-if="isManualRouteRecording && manualRouteNodes.length >= 2"
  class="flow-actions"
>
      <button type="button" @click="createManualRouteFromSelection('supply1')">
        Manual avanço 1
      </button>

      <button type="button" @click="createManualRouteFromSelection('supply2')">
        Manual avanço 2
      </button>

      <button type="button" @click="createManualRouteFromSelection('supply3')">
        Manual avanço 3
      </button>
    </div>

    <div
  v-if="isManualRouteRecording && manualRouteNodes.length >= 2"
  class="flow-actions flow-actions--secondary"
>
      <button type="button" @click="createManualRouteFromSelection('return1')">
        Manual retorno 1
      </button>

      <button type="button" @click="createManualRouteFromSelection('return2')">
        Manual retorno 2
      </button>

      <button type="button" @click="createManualRouteFromSelection('return3')">
        Manual retorno 3
      </button>
    </div>

    <div class="flow-actions flow-actions--single">
      <button
        type="button"
        class="flow-button--primary"
        @click="saveCurrentRoute"
      >
        Guardar caminho
      </button>
    </div>

    <div v-if="hasLoadedModel && savedRoutes.length" class="saved-routes">
      <p class="connection-note">
        Caminhos guardados: {{ savedRoutes.length }}
      </p>

      <div class="flow-actions flow-actions--single">
        <button type="button" @click="createRouteGroupFromSelection">
          Criar grupo com selecionados
        </button>
      </div>

      <div
        v-for="route in savedRoutes"
        :key="route.id"
        class="saved-route-item"
      >
        <label class="saved-route-select">
          <input
            v-model="selectedRouteIds"
            type="checkbox"
            :value="route.id"
          />

          <span>{{ route.name }}</span>
        </label>

        <div>
          <button type="button" @click="applySavedRoute(route)">
            Aplicar
          </button>

          <button
            v-if="!route.hidden"
            type="button"
            @click="setSavedRouteVisibility(route.id, false)"
          >
            Ocultar
          </button>

          <button
            v-else
            type="button"
            @click="setSavedRouteVisibility(route.id, true)"
          >
            Mostrar
          </button>

          <button type="button" @click="reverseSavedRoute(route.id)">
            Inverter
          </button>

          <button type="button" @click="renameSavedRoute(route.id)">
            Renomear
          </button>

          <button type="button" @click="deleteSavedRoute(route.id)">
            Apagar
          </button>
        </div>
      </div>
    </div>

    <div v-if="hasLoadedModel && savedRouteGroups.length" class="saved-routes">
      <p class="connection-note">
        Grupos de caminhos: {{ savedRouteGroups.length }}
      </p>

      <div
        v-for="group in savedRouteGroups"
        :key="group.id"
        class="saved-route-item saved-route-item--group"
      >
        <div class="saved-route-group-info">
          <strong>{{ group.name }}</strong>

          <p class="saved-route-group-count">
            {{ getRoutesFromGroup(group).length }} caminho(s)
          </p>

          <ul class="saved-route-group-list">
            <li
              v-for="routeName in getRouteNamesFromGroup(group)"
              :key="routeName"
            >
              {{ routeName }}
            </li>
          </ul>
        </div>

        <div>
          <button type="button" @click="applyRouteGroup(group)">
            Aplicar
          </button>

          <button
            v-if="!group.hidden"
            type="button"
            @click="setSavedRouteGroupVisibility(group.id, false)"
          >
            Ocultar
          </button>

          <button
            v-else
            type="button"
            @click="setSavedRouteGroupVisibility(group.id, true)"
          >
            Mostrar
          </button>

          <button type="button" @click="reverseRouteGroup(group.id)">
            Inverter
          </button>

          <button type="button" @click="renameRouteGroup(group.id)">
            Renomear
          </button>

          <button type="button" @click="deleteRouteGroup(group.id)">
            Apagar
          </button>
        </div>
      </div>
    </div>

    <p class="connection-note">Inicio: {{ routeStartLabel }}</p>
    <p class="connection-note">Passagens: {{ routeWaypoints.length }}</p>
    <p class="connection-note">Fim: {{ routeEndLabel }}</p>


<p v-if="routeWarningMessage" class="route-warning-message">
  {{ routeWarningMessage }}
</p>


    <dl class="flow-stats">
      <div>
        <dt>Av. 1</dt>
        <dd>{{ pipeStats.supply1 }}</dd>
      </div>

      <div>
        <dt>Av. 2</dt>
        <dd>{{ pipeStats.supply2 }}</dd>
      </div>

      <div>
        <dt>Av. 3</dt>
        <dd>{{ pipeStats.supply3 }}</dd>
      </div>

      <div>
        <dt>Ret. 1</dt>
        <dd>{{ pipeStats.return1 }}</dd>
      </div>

      <div>
        <dt>Ret. 2</dt>
        <dd>{{ pipeStats.return2 }}</dd>
      </div>

      <div>
        <dt>Ret. 3</dt>
        <dd>{{ pipeStats.return3 }}</dd>
      </div>

      <div>
        <dt>Total</dt>
        <dd>{{ pipeStats.total }}</dd>
      </div>

      <div>
        <dt>Lig.</dt>
        <dd>{{ flowConnections.length }}</dd>
      </div>

      <div>
        <dt>Bloq.</dt>
        <dd>{{ blockedCount }}</dd>
      </div>
    </dl>

    <p class="flow-note">{{ flowMessage }}</p>
  </div>
</section>

      <section
        :class="[
          'flow-panel',
          isSimulationControlPanelMinimized ? 'flow-panel--minimized' : ''
        ]"
        aria-label="Simulation control"
      >
        <div class="flow-panel__header">
          <div>
            <p class="flow-panel__eyebrow">Simulação</p>
            <h2>Controlo</h2>
          </div>

          <span :class="['flow-status', isFlowing ? 'flow-status--on' : '']">
            {{ isCentralSimulationRunning ? 'SIM' : isFlowing ? 'ON' : 'OFF' }}
          </span>

          <button
            type="button"
            class="flow-panel__toggle"
            @click="toggleSimulationControlPanelMinimized"
          >
            {{ isSimulationControlPanelMinimized ? '+' : '−' }}
          </button>
        </div>

        <div class="flow-panel__content">
          <p class="selection-count">Selecionados: {{ selectedCount }}</p>

          <div class="flow-section-title">
            Válvulas e bloqueios
          </div>

          <div class="flow-actions flow-actions--secondary">
            <button type="button" @click="closeSelectedValves">
              Fechar válvula
            </button>

            <button type="button" @click="openSelectedValves">
              Abrir válvula
            </button>

            <button type="button" @click="clearBlockedPipes">
              Limpar bloqueios
            </button>
          </div>

          <div class="flow-section-title">
            Animação
          </div>

          <div class="flow-actions flow-actions--secondary">
  <button type="button" @click="toggleFlow">
    {{ isFlowing ? 'Pausar' : 'Animar' }}
  </button>

  <button type="button" @click="rebuildManualFlowLayer">
    Atualizar
  </button>
</div>

          <label class="flow-slider">
            <span>Velocidade</span>
            <input
              v-model.number="flowSpeed"
              type="range"
              min="0.2"
              max="3"
              step="0.1"
            />
          </label>

          <div class="flow-actions flow-actions--single">
            <button type="button" @click="toggleCentralSimulation">
              {{ isCentralSimulationRunning ? 'Parar simulação' : 'Simular central' }}
            </button>
          </div>

          <p class="connection-note">
            Bloqueios ativos: {{ blockedCount }}
          </p>
        </div>
      </section>
    </div>
  </div>

  <a
  href="https://github.com/bastto"
  target="_blank"
  rel="noopener noreferrer"
  class="corner-logo"
>
  /src/assets/bastto-logo.svg
</a>
  </template>

<script setup lang="ts">
import { onBeforeUnmount, onMounted, reactive, ref } from "vue";
import * as THREE from "three";
import * as OBC from "@thatopen/components";
import * as FRAGS from "@thatopen/fragments";
import * as BUI from "@thatopen/ui";
import * as BUIC from "@thatopen/ui-obc";
import * as OBCF from "@thatopen/components-front";

type PipeCircuit =
  | "supply1"
  | "supply2"
  | "supply3"
  | "return1"
  | "return2"
  | "return3";
type SelectionMap = Map<string, Set<number>>;
type MepElementType =
  | "pipe"
  | "isolationValve"
  | "normallyOpenValve"
  | "normallyClosedValve"
  | "collector"
  | "booster"
  | "reservoirWithResistance"
  | "reservoirWithoutResistance";

type CircuitType =
  | "hotWater"
  | "coldWater"
  | "heating"
  | "cooling"
  | "domesticHotWater"
  | "domesticColdWater"
  | "return"
  | "unknown";

type MepElement = {
  modelId: string;
  localId: number;
  elementType: MepElementType;
  circuitType: CircuitType;
  name?: string;
  category?: string;
  objectType?: string;
  tag?: string;
  state?: "open" | "closed" | "on" | "off";
};

type PipeParticle = {
  mesh: any;
  start: any;
  end: any;
  offset: number;
  length: number;
};

type FlowNode = {
  modelId: string;
  localId: number;
};

type SavedRoute = {
  id: string;
  name: string;
  temperature: PipeCircuit;
  path: FlowNode[];
  hidden?: boolean;
};

type SavedRouteGroup = {
  id: string;
  name: string;
  temperature: PipeCircuit;
  routeIds: string[];
  hidden?: boolean;
};

type RouteDirectionAppearance = {
  routeName: string;
  previousKey: string | null;
  nextKey: string | null;
};

type SharedRouteDirectionConflict = {
  node: FlowNode;
  routeNames: string[];
};

type SavedReversedDirection = {
  modelId: string;
  localIds: number[];
};

type FlowConnection = {
  from: FlowNode;
  to: FlowNode;
  temperature: PipeCircuit;
};

type StaticFlowObject = {
  object: any;
};

type PipeDirectionHints = {
  upstream?: any;
  downstream?: any;
};

type PipeGraphItem = FlowNode & {
  box: any;
  center: any;
  endpoints: any[];
};

const containerRef = ref<HTMLDivElement | null>(null);
const ifcInput = ref<HTMLInputElement | null>(null);
const isLoading = ref(false);
const isFlowing = ref(false);
const isCentralSimulationRunning = ref(false);
const isManualFlowAnimationRunning = ref(false);
const isFlowManuallyPaused = ref(false);
const loadingProgress = ref(0);
const loadingFileName = ref("");
const flowSpeed = ref(1);
const flowMessage = ref("Seleciona tubos no modelo e atribui um circuito.");
const routeWarningMessage = ref("");
const selectedCount = ref(0);
const selectedMepElementInfo = ref("Nenhum elemento classificado selecionado.");
const hasLoadedModel = ref(false);
const isElementPanelMinimized = ref(true);
const isFlowControlsPanelMinimized = ref(true);
const isSimulationControlPanelMinimized = ref(true);
const routeStartLabel = ref("nenhum");
const routeEndLabel = ref("nenhum");
const blockedCount = ref(0);
const pipeStats = reactive({
  supply: 0,
  return: 0,

  supply1: 0,
  supply2: 0,
  supply3: 0,

  return1: 0,
  return2: 0,
  return3: 0,

  total: 0,
});

const MEP_ELEMENTS_STORAGE_KEY = "bastto-viewer-mep-elements";
const ROUTES_STORAGE_KEY = "bastto-viewer-routes";
const REVERSED_DIRECTIONS_STORAGE_KEY =
  "bastto-viewer-reversed-directions";
const ROUTE_GROUPS_STORAGE_KEY =
  "bastto-viewer-route-groups";
const HIDDEN_FLOW_ARROWS_STORAGE_KEY =
  "bastto-viewer-hidden-flow-arrows";
const SYNCED_PIPE_DIRECTIONS_STORAGE_KEY =
  "bastto-viewer-synced-pipe-directions";

let world: any;
let serializer: FRAGS.IfcImporter;
let fragmentManager: OBC.FragmentsManager;
let fragmentBytes: ArrayBuffer | null = null;
let animationFrame = 0;

const loadedModels = new Map<string, FRAGS.FragmentsModel>();
const flowGroup = new THREE.Group();
const pipeParticles: PipeParticle[] = [];
const staticFlowObjects: StaticFlowObject[] = [];
const selectedItems: SelectionMap = new Map();
const flowConnections = reactive<FlowConnection[]>([]);
const routeWaypoints = reactive<FlowNode[]>([]);
const manualRouteNodes = reactive<FlowNode[]>([]);
const isManualRouteRecording = ref(false);
const mepElements = reactive<Record<string, MepElement>>({});
const savedRoutes = reactive<SavedRoute[]>([]);
const savedRouteGroups = reactive<SavedRouteGroup[]>([]);
const selectedRouteIds = ref<string[]>([]);
const activeRouteGroupId = ref<string | null>(null);
let routeStart: FlowNode | null = null;
let routeEnd: FlowNode | null = null;
const blockedPipes: SelectionMap = new Map();
const valveBlockedPipeLinks = new Map<string, FlowNode[]>();
const manualAssignments: Record<PipeCircuit, SelectionMap> = {
  supply1: new Map(),
  supply2: new Map(),
  supply3: new Map(),
  return1: new Map(),
  return2: new Map(),
  return3: new Map(),
};
const reversedPipeDirections: SelectionMap = new Map();
const syncedPipeDirections: SelectionMap = new Map();
const hiddenFlowArrowElements: SelectionMap = new Map();

const mepElementHighlightColors: Record<MepElementType, number> = {
  pipe: 0x00ffff,
  isolationValve: 0x0000ff,
  normallyOpenValve: 0x00ff00,
  normallyClosedValve: 0xff0000,
  collector: 0xff00ff,
  booster: 0x00ff00,
  reservoirWithResistance: 0xffff00,
  reservoirWithoutResistance: 0xff6600,
};

const circuitMaterials = {
  supply1: new THREE.MeshBasicMaterial({
    color: 0xff0000,
    transparent: true,
    opacity: 0.9,
    depthTest: false,
  }),

  supply2: new THREE.MeshBasicMaterial({
    color: 0xff5252,
    transparent: true,
    opacity: 0.9,
    depthTest: false,
  }),

  supply3: new THREE.MeshBasicMaterial({
    color: 0xff8a80,
    transparent: true,
    opacity: 0.9,
    depthTest: false,
  }),

  return1: new THREE.MeshBasicMaterial({
    color: 0xff8c00,
    transparent: true,
    opacity: 0.9,
    depthTest: false,
  }),

  return2: new THREE.MeshBasicMaterial({
    color: 0xffa726,
    transparent: true,
    opacity: 0.9,
    depthTest: false,
  }),

  return3: new THREE.MeshBasicMaterial({
    color: 0xffc107,
    transparent: true,
    opacity: 0.9,
    depthTest: false,
  }),
};

onMounted(async () => {
  if (!containerRef.value) return;

  BUI.Manager.init();

  const components = new OBC.Components();
  const worlds = components.get(OBC.Worlds);
  world = worlds.create<OBC.SimpleScene, OBC.SimpleCamera, OBC.SimpleRenderer>();

  world.scene = new OBC.SimpleScene(components);
  world.scene.three.background = null;

  const viewport = document.createElement("bim-viewport");
  world.renderer = new OBC.SimpleRenderer(components, viewport);
  world.camera = new OBC.OrthoPerspectiveCamera(components);

  components.init();
  world.camera.controls.setLookAt(-60, 30, -20, 0, 0, -20);

  world.scene.setup();
  world.scene.three.add(flowGroup);

  const grids = components.get(OBC.Grids);
  grids.create(world);

  serializer = new FRAGS.IfcImporter();
  serializer.wasm = {
    absolute: true,
    path: "https://unpkg.com/web-ifc@0.0.69/",
  };

  const ifcLoader = components.get(OBC.IfcLoader);
  await ifcLoader.setup({
    autoSetWasm: false,
    wasm: {
      path: "https://unpkg.com/web-ifc@0.0.69/",
      absolute: true,
    },
  });

  fragmentManager = components.get(OBC.FragmentsManager);
  fragmentManager.init("/worker.mjs");

  world.camera.controls.addEventListener("rest", async () => {
  await fragmentManager.core.update(true);

  if (countAssignments() > 0) {
    await rebuildManualFlowLayer();
  }
});

  fragmentManager.list.onItemSet.add(async ({ value: model }) => {
  model.useCamera(world?.camera.three as any);
  world?.scene.three.add(model.object);
  loadedModels.set(model.modelId, model);

  hasLoadedModel.value = true;

  await fragmentManager.core.update(true);

  if (savedRoutes.length) {
    await applyAllSavedRoutes();
  } else {
    flowMessage.value =
      `Modelo carregado: ${model.modelId}. Seleciona tubos e atribui os circuitos.`;
  }
});

  loadMepElementsFromStorage();
loadRoutesFromStorage();
loadRouteGroupsFromStorage();
loadReversedDirectionsFromStorage();
loadSyncedPipeDirectionsFromStorage();
loadHiddenFlowArrowsFromStorage();

  createBimPanel(components, viewport);
  animateFlow();
});

onBeforeUnmount(() => {
  cancelAnimationFrame(animationFrame);
  clearFlowLayer();
});

function createBimPanel(components: OBC.Components, viewport: HTMLElement) {
  const [modelsList] = BUIC.tables.modelsList({
    components,
    metaDataTags: ["schema"],
    actions: { download: true },
  });

  const [spatialTree] = BUIC.tables.spatialTree({
    components,
    models: [],
  });

  const [propertiesTable, updatePropertiesTable] = BUIC.tables.itemsData({
    components,
    modelIdMap: {},
  });

  const highlighter = components.get(OBCF.Highlighter);
  highlighter.setup({ world });
  highlighter.styles.set("select", {
  color: new THREE.Color(0x00ff66),
  opacity: 0.85,
  transparent: true,
  renderedFaces: FRAGS.RenderedFaces.TWO,
});

  highlighter.events.select.onHighlight.add(async (modelIdMap) => {
  replaceSelection(modelIdMap);
  updatePropertiesTable({ modelIdMap });

  await showSelectedMepElementInfo();

  addSelectedNodeToManualRoute();

  if (countAssignments() > 0) {
    await rebuildManualFlowLayer();
  }
});

  highlighter.events.select.onClear.add(async () => {
  selectedItems.clear();
  selectedCount.value = 0;
  selectedMepElementInfo.value = "Nenhum elemento classificado selecionado.";

  updatePropertiesTable({
    modelIdMap: {},
  });

  if (countAssignments() > 0) {
    await rebuildManualFlowLayer();
  }
});

  propertiesTable.preserveStructureOnFilter = true;
  propertiesTable.indentationInText = false;

  const panel = BUI.Component.create(() => {
    const [loadFragBtn] = BUIC.buttons.loadFrag({ components });

    const onSearchSpatialTree = (e: Event) => {
      const input = e.target as BUI.TextInput;
      spatialTree.queryString = input.value;
    };

    const onTextInput = (e: Event) => {
      const input = e.target as BUI.TextInput;
      propertiesTable.queryString = input.value !== "" ? input.value : null;
    };

    return BUI.html`
      <bim-panel label="IFC Models">
        <bim-panel-section label="Importing">
          ${loadFragBtn}
          <bim-button label="Load IFC" @click=${openIfcDialog}></bim-button>
          <bim-text-input @input=${onSearchSpatialTree} placeholder="Search..." debounce="200"></bim-text-input>
          ${spatialTree}
        </bim-panel-section>
        <bim-panel-section icon="mage:box-3d-fill" label="Loaded Models">
          ${modelsList}
        </bim-panel-section>
        <bim-panel-section label="Properties">
          <bim-text-input @input=${onTextInput} placeholder="Search Property" debounce="200"></bim-text-input>
          ${propertiesTable}
        </bim-panel-section>
      </bim-panel>
    `;
  });

  const app = document.getElementById("appGrid") as BUI.Grid<["main"]>;
  app.layouts = {
    main: {
      template: `
        "panel viewport"
        / 23rem 1fr
      `,
      elements: { panel, viewport },
    },
  };

  app.layout = "main";
}

const openIfcDialog = () => ifcInput.value?.click();

const convertIFC = async (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;

  isLoading.value = true;
  loadingProgress.value = 0;
  loadingFileName.value = file.name;
  clearFlowLayer();
  resetAssignmentMaps();

  try {
    const buffer = await file.arrayBuffer();
    const ifcBytes = new Uint8Array(buffer);

    fragmentBytes = await serializer.process({
      bytes: ifcBytes,
      progressCallback: (progress: number) => {
        loadingProgress.value = Math.round(progress * 100);
      },
    });

    if (!fragmentBytes) return;
    await fragmentManager.core.load(fragmentBytes, { modelId: file.name });
  } catch (err) {
    console.error("IFC conversion failed:", err);
    alert("Failed to convert IFC file. Please try again.");
  } finally {
    isLoading.value = false;
  }
};

async function analyseLoadedModels() {
  if (!loadedModels.size) {
    flowMessage.value = "Carrega primeiro um IFC ou um ficheiro .frag.";
    return;
  }

  clearFlowLayer();
  isLoading.value = true;
  loadingFileName.value = "pipe analysis";
  loadingProgress.value = 0;

  try {
    for (const model of loadedModels.values()) {
      await analyseModelPipes(model);
    }

    isFlowing.value = false;
    flowMessage.value =
  "Modelo carregado. Define manualmente os circuitos de avanço e retorno.";
    await fragmentManager.core.update(true);
  } catch (error) {
    console.error("Pipe analysis failed:", error);
    flowMessage.value = "Nao foi possivel analisar as tubagens deste modelo.";
  } finally {
    isLoading.value = false;
  }
}

async function analyseModelPipes(model: FRAGS.FragmentsModel) {
  const categories = await model.getItemsOfCategories([
    /IFCFLOWSEGMENT/i,
    /IFCPIPESEGMENT/i,
    /IFCPIPEFITTING/i,
    /IFCFLOWFITTING/i,
    /IFCFLOWCONTROLLER/i,
  ]);
  const localIds = [...new Set(Object.values(categories).flat())];
  if (!localIds.length) return;

  const chunks = chunk(localIds, 80);

  for (let index = 0; index < chunks.length; index++) {
    const ids = chunks[index];
    const data = await model.getItemsData(ids, {
      attributesDefault: true,
      relationsDefault: { attributes: true, relations: false },
    });
    const boxes = await model.getBoxes(ids);

    const supplyIds: number[] = [];
const returnIds: number[] = [];

ids.forEach((id, itemIndex) => {
  const temperature = classifyPipe(data[itemIndex]);

  if (temperature === "supply1") {
    supplyIds.push(id);
  } else {
    returnIds.push(id);
  }

  const box = boxes[itemIndex];

  if (box) {
    addPipeParticles(box, temperature);
  }
});

    if (supplyIds.length)
  await model.highlight(
    supplyIds,
    createHighlight(0xff3b30, "supply1")
  );

if (returnIds.length)
  await model.highlight(
    returnIds,
    createHighlight(0xffc107, "return1")
  );

    pipeStats.supply += supplyIds.length;
pipeStats.return += returnIds.length;
    pipeStats.total += ids.length;
    loadingProgress.value = Math.round(((index + 1) / chunks.length) * 100);
  }
}

async function assignSelectedPipes(circuit: PipeCircuit) {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro um ou mais tubos no modelo.";
    return;
  }

  for (const [modelId, ids] of selectedItems) {
  const targetSet = getAssignmentSet(circuit, modelId);

  for (const id of ids) {

    // remove o tubo de todos os outros circuitos
    Object.values(manualAssignments).forEach((setMap) => {
      setMap.get(modelId)?.delete(id);
    });

    targetSet.add(id);
  }
}

  updateManualStats();
  await rebuildManualFlowLayer();
  flowMessage.value =
  `${selectedCount.value} elemento(s) marcados como ${circuit}.`;
}

async function rebuildManualFlowLayer() {
  const wasFlowingBeforeRebuild = isFlowing.value;

clearFlowVisuals(true);

  const hasAssignments = countAssignments() > 0;
  const hasConnections = flowConnections.length > 0;

  if (!hasAssignments && !hasConnections) {
    flowMessage.value = "Ainda nao ha tubos marcados.";
    return;
  }

  await addAssignmentsToScene("supply1");
await addAssignmentsToScene("supply2");
await addAssignmentsToScene("supply3");

await addAssignmentsToScene("return1");
await addAssignmentsToScene("return2");
await addAssignmentsToScene("return3");

  updateManualStats();

  const shouldKeepAnimating =
  !isFlowManuallyPaused.value &&
  (isManualFlowAnimationRunning.value || isCentralSimulationRunning.value);

isFlowing.value = shouldKeepAnimating && pipeParticles.length > 0;

  await fragmentManager.core.update(true);

}

async function addAssignmentsToScene(temperature: PipeCircuit) {
  for (const [modelId, idsSet] of manualAssignments[temperature]) {
    const model = loadedModels.get(modelId);
    const allIds = [...idsSet];

if (!model || !allIds.length) continue;

const hiddenIds = allIds.filter((localId) =>
  isNodeHiddenBySavedRouteVisibility({ modelId, localId }),
);

const ids = allIds.filter(
  (localId) => !isNodeHiddenBySavedRouteVisibility({ modelId, localId }),
);

if (hiddenIds.length) {
  await model.resetHighlight(hiddenIds);
}

if (!ids.length) continue;

    const circuitColors = {
  supply1: 0xff0000,
  supply2: 0xff5252,
  supply3: 0xff8a80,
  return1: 0xff8c00,
  return2: 0xffa726,
  return3: 0xffc107,
};

await model.highlight(
  ids,
  createHighlight(
    circuitColors[temperature],
    temperature,
  ),
);

    const boxes = await model.getBoxes(ids);
    for (let index = 0; index < boxes.length; index++) {
      const box = boxes[index];
      if (!box) continue;

      const localId = ids[index];

if (isPipeBlocked(modelId, localId)) {
  continue;
}

if (isFlowArrowHidden(modelId, localId)) {
  continue;
}

const hints = await getPipeDirectionHints({ modelId, localId });

addPipeParticles(
  box,
  temperature,
  hints,
  { modelId, localId },
);
    }
  }
}

async function clearSelectedManualAssignments() {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro um ou mais elementos para limpar a marca.";
    return;
  }

  let clearedCount = 0;
  const idsByModel = new Map<string, number[]>();

  for (const [modelId, ids] of selectedItems) {
    for (const localId of ids) {

      getAssignmentSet("supply1", modelId).delete(localId);
      getAssignmentSet("supply2", modelId).delete(localId);
      getAssignmentSet("supply3", modelId).delete(localId);

      getAssignmentSet("return1", modelId).delete(localId);
      getAssignmentSet("return2", modelId).delete(localId);
      getAssignmentSet("return3", modelId).delete(localId);

      reversedPipeDirections.get(modelId)?.delete(localId);

      if (reversedPipeDirections.get(modelId)?.size === 0) {
        reversedPipeDirections.delete(modelId);
      }

      if (!idsByModel.has(modelId)) {
        idsByModel.set(modelId, []);
      }

      idsByModel.get(modelId)?.push(localId);

      clearedCount++;
    }
  }

  for (const [modelId, ids] of idsByModel) {
    const model = loadedModels.get(modelId);

    if (model && ids.length) {
      await model.resetHighlight(ids);
    }
  }

  saveReversedDirectionsToStorage();
  updateManualStats();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    clearFlowVisuals();
    await fragmentManager.core.update(true);
  }

  flowMessage.value = clearedCount
    ? `${clearedCount} marca(s) selecionada(s) removida(s).`
    : "Nenhuma marca selecionada para remover.";
}

async function clearManualAssignments() {
  clearFlowVisuals();

  for (const [modelId, model] of loadedModels) {
    const ids = [
  ...getAssignmentSet("supply1", modelId),
  ...getAssignmentSet("supply2", modelId),
  ...getAssignmentSet("supply3", modelId),
  ...getAssignmentSet("return1", modelId),
  ...getAssignmentSet("return2", modelId),
  ...getAssignmentSet("return3", modelId),
];
    if (ids.length) await model.resetHighlight(ids);
  }

  manualAssignments.supply1.clear();
manualAssignments.supply2.clear();
manualAssignments.supply3.clear();

manualAssignments.return1.clear();
manualAssignments.return2.clear();
manualAssignments.return3.clear();

reversedPipeDirections.clear();
saveReversedDirectionsToStorage();

  flowConnections.splice(0);
  routeWaypoints.splice(0);
  routeStart = null;
  routeEnd = null;
  routeStartLabel.value = "nenhum";
  routeEndLabel.value = "nenhum";
  updateManualStats();
  flowMessage.value =
  "Marcações removidas. Seleciona novos tubos para definir um circuito.";
  await fragmentManager.core.update(true);
}

function resetAssignmentMaps() {
  manualAssignments.supply1.clear();
manualAssignments.supply2.clear();
manualAssignments.supply3.clear();

manualAssignments.return1.clear();
manualAssignments.return2.clear();
manualAssignments.return3.clear();
  flowConnections.splice(0);
  routeWaypoints.splice(0);
  routeStart = null;
  routeEnd = null;
  routeStartLabel.value = "nenhum";
  routeEndLabel.value = "nenhum";
}

function startManualRouteRecording() {
  manualRouteNodes.splice(0);
  isManualRouteRecording.value = true;
  flowMessage.value =
    "Modo manual ativo. Seleciona os tubos pela ordem do percurso.";
}

function cancelManualRouteRecording() {
  manualRouteNodes.splice(0);
  isManualRouteRecording.value = false;
  flowMessage.value = "Caminho manual cancelado.";
}

function toggleManualRouteRecording() {
  if (isManualRouteRecording.value) {
    cancelManualRouteRecording();
    return;
  }

  startManualRouteRecording();
}

function addSelectedNodeToManualRoute() {
  if (!isManualRouteRecording.value) {
    return;
  }

  const node = getFirstSelectedNode();

  if (!node) {
    return;
  }

  const lastNode = manualRouteNodes[manualRouteNodes.length - 1];

  if (lastNode && isSameNode(lastNode, node)) {
    return;
  }

  manualRouteNodes.push(node);

  flowMessage.value =
    `Ponto manual adicionado. Total: ${manualRouteNodes.length}.`;
}

function setRouteStart() {
  const node = getFirstSelectedNode();
  if (!node) {
    flowMessage.value = "Seleciona primeiro o tubo/ponto inicial.";
    return;
  }

  routeStart = node;
  routeStartLabel.value = formatNodeLabel(node);
  flowMessage.value = "Inicio definido. Seleciona o ponto final e carrega em Definir fim.";
}

function setRouteEnd() {
  const node = getFirstSelectedNode();
  if (!node) {
    flowMessage.value = "Seleciona primeiro o tubo/ponto final.";
    return;
  }

  routeEnd = node;
  routeEndLabel.value = formatNodeLabel(node);
  flowMessage.value =
  "Fim definido. Agora calcula o caminho de avanço ou retorno.";
}

function addRouteWaypoint() {
  const node = getFirstSelectedNode();
  if (!node) {
    flowMessage.value = "Seleciona primeiro um tubo/ponto por onde o caminho deve passar.";
    return;
  }

  const alreadyExists = routeWaypoints.some((waypoint) => isSameNode(waypoint, node));
  if (!alreadyExists) routeWaypoints.push(node);

  flowMessage.value = `Ponto de passagem adicionado. Total: ${routeWaypoints.length}.`;
}

async function createManualRouteFromSelection(temperature: PipeCircuit) {
  if (manualRouteNodes.length < 2) {
    flowMessage.value =
      "Seleciona pelo menos dois tubos pela ordem do caminho manual.";
    return;
  }

  const path = [...manualRouteNodes];

  flowConnections.splice(0);

  for (let index = 0; index < path.length - 1; index++) {
    flowConnections.push({
      from: path[index],
      to: path[index + 1],
      temperature,
    });
  }

  assignPathToTemperature(path, temperature);

  updateManualStats();

  await rebuildManualFlowLayer();

  manualRouteNodes.splice(0);
  isManualRouteRecording.value = false;

  flowMessage.value =
    `Caminho manual ${getCircuitLabel(temperature)} criado com ${path.length} tubo(s).`;
}

async function createAutoRoute(temperature: PipeCircuit) {
  if (!routeStart || !routeEnd) {
    flowMessage.value = "Define primeiro o inicio e o fim do caminho.";
    return;
  }

  if (
    routeStart.modelId === routeEnd.modelId &&
    routeStart.localId === routeEnd.localId
  ) {
    flowMessage.value = "O inicio e o fim precisam de ser pontos diferentes.";
    return;
  }

  if (routeStart.modelId !== routeEnd.modelId) {
    flowMessage.value = "O caminho automatico ainda so funciona dentro do mesmo modelo.";
    return;
  }

  isLoading.value = true;
  loadingFileName.value = "route calculation";
  loadingProgress.value = 0;

  try {
    const checkpoints = [routeStart, ...routeWaypoints, routeEnd];
    const path = await findRouteThroughCheckpoints(checkpoints);

    if (!path.length) {
      flowMessage.value = "Nao encontrei um caminho continuo entre o inicio e o fim.";
      return;
    }

    flowConnections.splice(0);
    for (let index = 0; index < path.length - 1; index++) {
      flowConnections.push({
        from: path[index],
        to: path[index + 1],
        temperature,
      });
    }

    assignPathToTemperature(path, temperature);
    updateManualStats();
    await rebuildManualFlowLayer();
    flowMessage.value =
  `Caminho ${temperature} criado com ${path.length} tubos.`;
  } catch (error) {
    console.error("Automatic route failed:", error);
    flowMessage.value = "Nao foi possivel calcular o caminho automatico.";
  } finally {
    isLoading.value = false;
  }
}

function assignPathToTemperature(
  path: FlowNode[],
  temperature: PipeCircuit,
) {
  for (const node of path) {

    Object.values(manualAssignments).forEach((setMap) => {
      setMap.get(node.modelId)?.delete(node.localId);
    });

    getAssignmentSet(temperature, node.modelId).add(node.localId);
  }
}

function saveCurrentRoute() {
  if (!flowConnections.length) {
    flowMessage.value = "Cria primeiro um caminho antes de o guardar.";
    return;
  }

  const temperature = flowConnections[0].temperature;

  const path: FlowNode[] = [
    flowConnections[0].from,
    ...flowConnections.map((connection) => connection.to),
  ];

  const circuitLabel = getCircuitLabel(temperature);
const routeNumber = getNextRouteNumberForCircuit(temperature);

savedRoutes.push({
  id: crypto.randomUUID(),
  name: `Caminho ${circuitLabel} - ${routeNumber}`,
  temperature,
  path,
});

  saveRoutesToStorage();

  flowMessage.value = "Caminho guardado com sucesso.";
}

async function deleteSavedRoute(routeId: string) {
  const index = savedRoutes.findIndex(
    (route) => route.id === routeId,
  );

  if (index === -1) return;

  const route = savedRoutes[index];

  savedRoutes.splice(index, 1);

  const idsByModel = new Map<string, number[]>();

  for (const node of route.path) {
    getAssignmentSet("supply1", node.modelId).delete(node.localId);
    getAssignmentSet("supply2", node.modelId).delete(node.localId);
    getAssignmentSet("supply3", node.modelId).delete(node.localId);

    getAssignmentSet("return1", node.modelId).delete(node.localId);
    getAssignmentSet("return2", node.modelId).delete(node.localId);
    getAssignmentSet("return3", node.modelId).delete(node.localId);

    reversedPipeDirections.get(node.modelId)?.delete(node.localId);

if (reversedPipeDirections.get(node.modelId)?.size === 0) {
  reversedPipeDirections.delete(node.modelId);
}

    if (!idsByModel.has(node.modelId)) {
      idsByModel.set(node.modelId, []);
    }

    idsByModel.get(node.modelId)?.push(node.localId);
  }

  for (const [modelId, ids] of idsByModel) {
    const model = loadedModels.get(modelId);

    if (model && ids.length) {
      await model.resetHighlight(ids);
    }
  }

  flowConnections.splice(0);

  saveRoutesToStorage();

  saveReversedDirectionsToStorage();

  updateManualStats();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    clearFlowVisuals();
    await fragmentManager.core.update(true);
  }

  flowMessage.value = "Caminho apagado.";
}

async function resetRoutePathHighlight(route: SavedRoute) {
  const idsByModel = new Map<string, number[]>();

  for (const node of route.path) {
    if (!idsByModel.has(node.modelId)) {
      idsByModel.set(node.modelId, []);
    }

    idsByModel.get(node.modelId)?.push(node.localId);
  }

  for (const [modelId, ids] of idsByModel) {
    const model = loadedModels.get(modelId);

    if (model && ids.length) {
      await model.resetHighlight(ids);
    }
  }
}

async function setSavedRouteVisibility(routeId: string, shouldShow: boolean) {
  const route = savedRoutes.find((savedRoute) => savedRoute.id === routeId);

  if (!route) return;

  route.hidden = !shouldShow;

  saveRoutesToStorage();

  await resetRoutePathHighlight(route);

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    await fragmentManager.core.update(true);
  }

  flowMessage.value = shouldShow
    ? `Caminho "${route.name}" visível.`
    : `Caminho "${route.name}" oculto.`;
}

async function applySavedRoute(route: SavedRoute) {
  activeRouteGroupId.value = null;
  const loadedModelIds = [...loadedModels.keys()];

  if (!loadedModelIds.length) {
    flowMessage.value = "Carrega primeiro o IFC antes de aplicar o caminho.";
    return;
  }

  const fallbackModelId = loadedModelIds[0];

  const adaptedPath = route.path.map((node) => {
    if (loadedModels.has(node.modelId)) {
      return node;
    }

    return {
      modelId: fallbackModelId,
      localId: node.localId,
    };
  });

  assignPathToTemperature(
    adaptedPath,
    route.temperature,
  );

  flowConnections.splice(0);

  for (let index = 0; index < adaptedPath.length - 1; index++) {
    flowConnections.push({
      from: adaptedPath[index],
      to: adaptedPath[index + 1],
      temperature: route.temperature,
    });
  }

  updateManualStats();

  await rebuildManualFlowLayer();

  flowMessage.value =
    `Caminho ${route.name} aplicado.`;
}

async function setSavedRouteGroupVisibility(
  groupId: string,
  shouldShow: boolean,
) {
  const group = savedRouteGroups.find(
    (savedGroup) => savedGroup.id === groupId,
  );

  if (!group) return;

  group.hidden = !shouldShow;

  saveRouteGroupsToStorage();

  const routes = getRoutesFromGroup(group);

  for (const route of routes) {
    await resetRoutePathHighlight(route);
  }

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    await fragmentManager.core.update(true);
  }

  flowMessage.value = shouldShow
    ? `Grupo "${group.name}" visível.`
    : `Grupo "${group.name}" oculto.`;
}

async function applyRouteGroup(group: SavedRouteGroup) {
  activeRouteGroupId.value = group.id;
  const routes = getRoutesFromGroup(group);

  if (!routes.length) {
    flowMessage.value = "Este grupo não tem caminhos válidos.";
    return;
  }

  const loadedModelIds = [...loadedModels.keys()];

  if (!loadedModelIds.length) {
    flowMessage.value = "Carrega primeiro o IFC antes de aplicar o grupo.";
    return;
  }

  const fallbackModelId = loadedModelIds[0];

  flowConnections.splice(0);

  for (const route of routes) {
    const adaptedPath = route.path.map((node) => {
      if (loadedModels.has(node.modelId)) {
        return node;
      }

      return {
        modelId: fallbackModelId,
        localId: node.localId,
      };
    });

    assignPathToTemperature(
      adaptedPath,
      route.temperature,
    );

    for (let index = 0; index < adaptedPath.length - 1; index++) {
      flowConnections.push({
        from: adaptedPath[index],
        to: adaptedPath[index + 1],
        temperature: route.temperature,
      });
    }
  }

  updateManualStats();

  await rebuildManualFlowLayer();

  const directionConflicts = findSharedRouteDirectionConflicts(group);

routeWarningMessage.value = directionConflicts.length
  ? `Atenção: ${directionConflicts.length} tubo(s) partilhado(s) podem ter sentidos diferentes.`
  : "";

flowMessage.value =
  `Grupo "${group.name}" aplicado com ${routes.length} caminho(s).`;
}

async function reverseRouteGroup(groupId: string) {
  const group = savedRouteGroups.find(
    (savedGroup) => savedGroup.id === groupId,
  );

  if (!group) return;

  const routes = getRoutesFromGroup(group);

  if (!routes.length) {
    flowMessage.value = "Este grupo não tem caminhos válidos.";
    return;
  }

  for (const route of routes) {
  const reversedPath = [...route.path].reverse();

  route.path.splice(
    0,
    route.path.length,
    ...reversedPath,
  );

  toggleSyncedPipesForPath(route.path);
}

saveRoutesToStorage();

  if (loadedModels.size) {
    await applyRouteGroup(group);
  }

  flowMessage.value =
    `Sentido do grupo "${group.name}" invertido.`;
}

function deleteRouteGroup(groupId: string) {
  const index = savedRouteGroups.findIndex(
    (group) => group.id === groupId,
  );

  if (index === -1) return;

  const groupName = savedRouteGroups[index].name;

  savedRouteGroups.splice(index, 1);

  saveRouteGroupsToStorage();

  flowMessage.value =
    `Grupo "${groupName}" apagado.`;
}

function renameRouteGroup(groupId: string) {
  const group = savedRouteGroups.find(
    (savedGroup) => savedGroup.id === groupId,
  );

  if (!group) return;

  const newName = prompt(
    "Novo nome do grupo:",
    group.name,
  );

  if (!newName) return;

  const trimmedName = newName.trim();

  if (!trimmedName) return;

  group.name = trimmedName;

  saveRouteGroupsToStorage();

  flowMessage.value =
    `Grupo renomeado para "${trimmedName}".`;
}

async function reverseSavedRoute(routeId: string) {
  const route = savedRoutes.find(
    (savedRoute) => savedRoute.id === routeId,
  );

  if (!route) return;

  const reversedPath = [...route.path].reverse();

route.path.splice(
  0,
  route.path.length,
  ...reversedPath,
);

toggleSyncedPipesForPath(route.path);

saveRoutesToStorage();

  const group = getGroupForRoute(routeId);

  if (loadedModels.size) {
    if (group) {
      await applyRouteGroup(group);
    } else {
      await applySavedRoute(route);
    }
  }

  flowMessage.value = group
    ? `Sentido do caminho "${route.name}" invertido dentro do grupo "${group.name}".`
    : `Sentido do caminho "${route.name}" invertido.`;
}

function createRouteGroupFromSelection() {
  if (selectedRouteIds.value.length < 2) {
    flowMessage.value =
      "Seleciona pelo menos dois caminhos para criar um grupo.";
    return;
  }

  const routes = savedRoutes.filter((route) =>
    selectedRouteIds.value.includes(route.id),
  );

  if (!routes.length) {
    flowMessage.value = "Nenhum caminho válido selecionado.";
    return;
  }

  const temperature = routes[0].temperature;

  const allSameCircuit = routes.every(
    (route) => route.temperature === temperature,
  );

  if (!allSameCircuit) {
    flowMessage.value =
      "Só podes agrupar caminhos do mesmo circuito.";
    return;
  }

  const circuitLabel = getCircuitLabel(temperature);

  const groupName = prompt(
    "Nome do grupo de caminhos:",
    `Grupo ${circuitLabel}`,
  );

  if (!groupName) return;

  const trimmedName = groupName.trim();

  if (!trimmedName) return;

  const newGroup: SavedRouteGroup = {
  id: crypto.randomUUID(),
  name: trimmedName,
  temperature,
  routeIds: [...selectedRouteIds.value],
};

savedRouteGroups.push(newGroup);

selectedRouteIds.value = [];

saveRouteGroupsToStorage();

const directionConflicts = findSharedRouteDirectionConflicts(newGroup);

routeWarningMessage.value = directionConflicts.length
  ? `Atenção: ${directionConflicts.length} tubo(s) partilhado(s) podem ter sentidos diferentes.`
  : "";

flowMessage.value =
  `Grupo "${trimmedName}" criado com ${routes.length} caminho(s).`;
}

function renameSavedRoute(routeId: string) {
  const route = savedRoutes.find(
    (savedRoute) => savedRoute.id === routeId,
  );

  if (!route) return;

  const newName = prompt(
    "Novo nome do caminho:",
    route.name,
  );

  if (!newName) return;

  const trimmedName = newName.trim();

  if (!trimmedName) return;

  route.name = trimmedName;

  saveRoutesToStorage();

  flowMessage.value =
    `Caminho renomeado para "${trimmedName}".`;
}

async function applyAllSavedRoutes() {
  if (!savedRoutes.length) {
    return;
  }

  const loadedModelIds = [...loadedModels.keys()];

  if (!loadedModelIds.length) {
    return;
  }

  const fallbackModelId = loadedModelIds[0];

  flowConnections.splice(0);

  let appliedCount = 0;

  for (const route of savedRoutes) {
    const adaptedPath = route.path.map((node) => {
      if (loadedModels.has(node.modelId)) {
        return node;
      }

      return {
        modelId: fallbackModelId,
        localId: node.localId,
      };
    });

    assignPathToTemperature(
      adaptedPath,
      route.temperature,
    );

    for (let index = 0; index < adaptedPath.length - 1; index++) {
      flowConnections.push({
        from: adaptedPath[index],
        to: adaptedPath[index + 1],
        temperature: route.temperature,
      });
    }

    appliedCount++;
  }

  updateManualStats();

  await rebuildManualFlowLayer();

  flowMessage.value =
    `${appliedCount} caminho(s) guardado(s) aplicado(s).`;
}

async function clearConnections() {
  flowConnections.splice(0);
  routeWaypoints.splice(0);
  routeStart = null;
  routeEnd = null;
  routeStartLabel.value = "nenhum";
  routeEndLabel.value = "nenhum";
  await rebuildManualFlowLayer();
  flowMessage.value = "Caminho automatico removido.";
}

async function addConnectionsToScene() {
  for (const connection of flowConnections) {
  if (isConnectionBlocked(connection)) {
    continue;
  }

  const fromCenter = await getNodeCenter(connection.from);
  const toCenter = await getNodeCenter(connection.to);
    if (!fromCenter || !toCenter) continue;

    addConnectionParticles(fromCenter, toCenter, connection.temperature);
    addConnectionLine(fromCenter, toCenter, connection.temperature);
  }
}

async function findRouteThroughCheckpoints(checkpoints: FlowNode[]) {
  const route: FlowNode[] = [];

  for (let index = 0; index < checkpoints.length - 1; index++) {
    const segment = await findPipePath(checkpoints[index], checkpoints[index + 1]);
    if (!segment.length) return [];

    if (route.length) segment.shift();
    route.push(...segment);
  }

  return route;
}

async function findPipePath(start: FlowNode, end: FlowNode): Promise<FlowNode[]> {
  const model = loadedModels.get(start.modelId);
  if (!model) return [];

  const graphItems = await getPipeGraphItems(model);
  const startKey = nodeKey(start);
  const endKey = nodeKey(end);

  if (!graphItems.has(startKey)) {
    const startItem = await getGraphItemForNode(model, start);
    if (startItem) graphItems.set(startKey, startItem);
  }

  if (!graphItems.has(endKey)) {
    const endItem = await getGraphItemForNode(model, end);
    if (endItem) graphItems.set(endKey, endItem);
  }

  if (!graphItems.has(startKey) || !graphItems.has(endKey)) return [];

  const graph = buildPipeGraph(graphItems);
  const pathKeys = shortestPath(graph, startKey, endKey);
  return pathKeys.map((key) => {
    const item = graphItems.get(key);
    return { modelId: item?.modelId ?? start.modelId, localId: item?.localId ?? 0 };
  });
}

async function getPipeGraphItems(model: FRAGS.FragmentsModel) {
  const categories = await model.getItemsOfCategories([
    /IFCFLOWSEGMENT/i,
    /IFCPIPESEGMENT/i,
    /IFCPIPEFITTING/i,
    /IFCFLOWFITTING/i,
    /IFCFLOWCONTROLLER/i,
  ]);
  const ids = [...new Set(Object.values(categories).flat())];
  const boxes = await model.getBoxes(ids);
  const items = new Map<string, PipeGraphItem>();

  for (let index = 0; index < ids.length; index++) {
    const item = graphItemFromBox(model.modelId, ids[index], boxes[index]);
    if (item) items.set(nodeKey(item), item);
  }

  return items;
}

async function getGraphItemForNode(model: FRAGS.FragmentsModel, node: FlowNode) {
  const [box] = await model.getBoxes([node.localId]);
  return graphItemFromBox(node.modelId, node.localId, box);
}

function graphItemFromBox(modelId: string, localId: number, box: any): PipeGraphItem | null {
  if (!box) return null;

  const size = new THREE.Vector3();
  const center = new THREE.Vector3();
  box.getSize(size);
  box.getCenter(center);

  const axis = getLongestAxis(size);
  const length = Math.max(size.getComponent(axis), 0.1);
  const endpointA = center.clone();
  const endpointB = center.clone();
  endpointA.setComponent(axis, center.getComponent(axis) - length / 2);
  endpointB.setComponent(axis, center.getComponent(axis) + length / 2);

  return {
    modelId,
    localId,
    box,
    center,
    endpoints: [endpointA, endpointB],
  };
}

function buildPipeGraph(items: Map<string, PipeGraphItem>) {
  const entries = [...items.entries()];
  const graph = new Map<string, Map<string, number>>();

  for (const [key] of entries) graph.set(key, new Map());

  for (let i = 0; i < entries.length; i++) {
    const [keyA, itemA] = entries[i];

    for (let j = i + 1; j < entries.length; j++) {
      const [keyB, itemB] = entries[j];
      const distance = pipeConnectionDistance(itemA, itemB);

      if (distance <= pipeConnectionTolerance(itemA, itemB)) {
        graph.get(keyA)?.set(keyB, distance);
        graph.get(keyB)?.set(keyA, distance);
      }
    }
  }

  return graph;
}

function pipeConnectionDistance(a: PipeGraphItem, b: PipeGraphItem) {
  let min = Number.POSITIVE_INFINITY;

  for (const endpointA of a.endpoints) {
    for (const endpointB of b.endpoints) {
      min = Math.min(min, endpointA.distanceTo(endpointB));
    }
  }

  return min;
}

function pipeConnectionTolerance(a: PipeGraphItem, b: PipeGraphItem) {
  const sizeA = new THREE.Vector3();
  const sizeB = new THREE.Vector3();
  a.box.getSize(sizeA);
  b.box.getSize(sizeB);

  const minA = Math.min(sizeA.x || 999, sizeA.y || 999, sizeA.z || 999);
  const minB = Math.min(sizeB.x || 999, sizeB.y || 999, sizeB.z || 999);
  const inferredDiameter = Math.min(minA, minB);

  return THREE.MathUtils.clamp(inferredDiameter * 2.5, 0.25, 0.9);
}

function shortestPath(graph: Map<string, Map<string, number>>, start: string, end: string) {
  const distances = new Map<string, number>();
  const previous = new Map<string, string | null>();
  const pending = new Set(graph.keys());

  for (const key of graph.keys()) {
    distances.set(key, key === start ? 0 : Number.POSITIVE_INFINITY);
    previous.set(key, null);
  }

  while (pending.size) {
    let current: string | null = null;
    let bestDistance = Number.POSITIVE_INFINITY;

    for (const key of pending) {
      const distance = distances.get(key) ?? Number.POSITIVE_INFINITY;
      if (distance < bestDistance) {
        current = key;
        bestDistance = distance;
      }
    }

    if (!current || bestDistance === Number.POSITIVE_INFINITY) break;
    if (current === end) break;

    pending.delete(current);

    for (const [neighbor, weight] of graph.get(current) ?? []) {
      if (!pending.has(neighbor)) continue;

      const candidate = bestDistance + weight;
      if (candidate < (distances.get(neighbor) ?? Number.POSITIVE_INFINITY)) {
        distances.set(neighbor, candidate);
        previous.set(neighbor, current);
      }
    }
  }

  if (!Number.isFinite(distances.get(end) ?? Number.POSITIVE_INFINITY)) return [];

  const path: string[] = [];
  let current: string | null = end;

  while (current) {
    path.unshift(current);
    current = previous.get(current) ?? null;
  }

  return path[0] === start ? path : [];
}

function nodeKey(node: FlowNode) {
  return `${node.modelId}:${node.localId}`;
}

function elementKey(modelId: string, localId: number) {
  return `${modelId}:${localId}`;
}

function saveMepElementsToStorage() {
  localStorage.setItem(
    MEP_ELEMENTS_STORAGE_KEY,
    JSON.stringify(mepElements),
  );
}

function saveRoutesToStorage() {
  localStorage.setItem(
    ROUTES_STORAGE_KEY,
    JSON.stringify(savedRoutes),
  );
}

function loadRoutesFromStorage() {
  const saved = localStorage.getItem(ROUTES_STORAGE_KEY);

  if (!saved) return;

  try {
    const parsed = JSON.parse(saved) as SavedRoute[];

    savedRoutes.splice(0);
    savedRoutes.push(...parsed);
  } catch (error) {
    console.error("Erro ao carregar caminhos guardados:", error);
  }
}

function saveRouteGroupsToStorage() {
  localStorage.setItem(
    ROUTE_GROUPS_STORAGE_KEY,
    JSON.stringify(savedRouteGroups),
  );
}

function loadRouteGroupsFromStorage() {
  const saved = localStorage.getItem(ROUTE_GROUPS_STORAGE_KEY);

  if (!saved) return;

  try {
    const parsed = JSON.parse(saved) as SavedRouteGroup[];

    savedRouteGroups.splice(0);
    savedRouteGroups.push(...parsed);
  } catch (error) {
    console.error("Erro ao carregar grupos de caminhos:", error);
  }
}

function loadMepElementsFromStorage() {
  const saved = localStorage.getItem(MEP_ELEMENTS_STORAGE_KEY);

  if (!saved) return;

  try {
    const parsed = JSON.parse(saved) as Record<string, MepElement>;

    for (const [key, element] of Object.entries(parsed)) {
      mepElements[key] = element;
    }
  } catch (error) {
    console.error("Erro ao carregar elementos MEP guardados:", error);
  }
}

function saveSyncedPipeDirectionsToStorage() {
  const data: SavedReversedDirection[] = [];

  for (const [modelId, ids] of syncedPipeDirections) {
    data.push({
      modelId,
      localIds: [...ids],
    });
  }

  localStorage.setItem(
    SYNCED_PIPE_DIRECTIONS_STORAGE_KEY,
    JSON.stringify(data),
  );
}

function loadSyncedPipeDirectionsFromStorage() {
  const saved = localStorage.getItem(SYNCED_PIPE_DIRECTIONS_STORAGE_KEY);

  if (!saved) return;

  try {
    const parsed = JSON.parse(saved) as SavedReversedDirection[];

    syncedPipeDirections.clear();

    for (const item of parsed) {
      syncedPipeDirections.set(
        item.modelId,
        new Set(item.localIds),
      );
    }
  } catch (error) {
    console.error("Erro ao carregar tubos sincronizados:", error);
  }
}

function saveReversedDirectionsToStorage() {
  const data: SavedReversedDirection[] = [];

  for (const [modelId, ids] of reversedPipeDirections) {
    data.push({
      modelId,
      localIds: [...ids],
    });
  }

  localStorage.setItem(
    REVERSED_DIRECTIONS_STORAGE_KEY,
    JSON.stringify(data),
  );
}

function loadReversedDirectionsFromStorage() {
  const saved = localStorage.getItem(REVERSED_DIRECTIONS_STORAGE_KEY);

  if (!saved) return;

  try {
    const parsed = JSON.parse(saved) as SavedReversedDirection[];

    reversedPipeDirections.clear();

    for (const item of parsed) {
      reversedPipeDirections.set(
        item.modelId,
        new Set(item.localIds),
      );
    }
  } catch (error) {
    console.error("Erro ao carregar sentidos invertidos:", error);
  }
}

function saveHiddenFlowArrowsToStorage() {
  const data: SavedReversedDirection[] = [];

  for (const [modelId, ids] of hiddenFlowArrowElements) {
    data.push({
      modelId,
      localIds: [...ids],
    });
  }

  localStorage.setItem(
    HIDDEN_FLOW_ARROWS_STORAGE_KEY,
    JSON.stringify(data),
  );
}

function loadHiddenFlowArrowsFromStorage() {
  const saved = localStorage.getItem(HIDDEN_FLOW_ARROWS_STORAGE_KEY);

  if (!saved) return;

  try {
    const parsed = JSON.parse(saved) as SavedReversedDirection[];

    hiddenFlowArrowElements.clear();

    for (const item of parsed) {
      hiddenFlowArrowElements.set(
        item.modelId,
        new Set(item.localIds),
      );
    }
  } catch (error) {
    console.error("Erro ao carregar setas ocultas:", error);
  }
}

function defineSelectedElementsAs(elementType: MepElementType) {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro um ou mais elementos no modelo.";
    return;
  }

  for (const [modelId, ids] of selectedItems) {
    for (const localId of ids) {
      const key = elementKey(modelId, localId);

      const defaultState =
  elementType === "normallyClosedValve"
    ? "closed"
    : elementType === "normallyOpenValve"
      ? "open"
      : mepElements[key]?.state;

mepElements[key] = {
  modelId,
  localId,
  elementType,
  circuitType: mepElements[key]?.circuitType ?? "unknown",
  state: defaultState,
};

if (elementType === "normallyClosedValve") {
  let blockedSet = blockedPipes.get(modelId);

  if (!blockedSet) {
    blockedSet = new Set<number>();
    blockedPipes.set(modelId, blockedSet);
  }

  blockedSet.add(localId);
}

if (elementType === "normallyOpenValve") {
  blockedPipes.get(modelId)?.delete(localId);

  if (blockedPipes.get(modelId)?.size === 0) {
    blockedPipes.delete(modelId);
  }
}
    }
  }

  saveMepElementsToStorage();

  flowMessage.value = `${selectedCount.value} elemento(s) definidos como ${getElementTypeLabel(elementType)}.`;
}

async function deleteSelectedElementDefinitions() {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro um ou mais elementos para apagar a definição.";
    return;
  }

  let deletedCount = 0;

  for (const [modelId, ids] of selectedItems) {
    const blockedSet = blockedPipes.get(modelId);

    for (const localId of ids) {
      const key = elementKey(modelId, localId);

      if (!mepElements[key]) {
        continue;
      }

      delete mepElements[key];

      if (blockedSet?.has(localId)) {
        blockedSet.delete(localId);
      }

      getAssignmentSet("supply1", modelId).delete(localId);
getAssignmentSet("supply2", modelId).delete(localId);
getAssignmentSet("supply3", modelId).delete(localId);

getAssignmentSet("return1", modelId).delete(localId);
getAssignmentSet("return2", modelId).delete(localId);
getAssignmentSet("return3", modelId).delete(localId);

      reversedPipeDirections.get(modelId)?.delete(localId);

if (reversedPipeDirections.get(modelId)?.size === 0) {
  reversedPipeDirections.delete(modelId);
}

      deletedCount++;
    }

    if (blockedSet && blockedSet.size === 0) {
      blockedPipes.delete(modelId);
    }
  }

  updateBlockedCount();
  updateManualStats();
  saveMepElementsToStorage();
  saveReversedDirectionsToStorage();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value = deletedCount
    ? `${deletedCount} definição(ões) apagada(s).`
    : "Os elementos selecionados não tinham definição guardada.";
}

function getAttributeValueText(value: any): string {
  if (value === null || value === undefined) {
    return "";
  }

  if (typeof value === "string" || typeof value === "number") {
    return String(value);
  }

  if (typeof value === "object") {
    if ("value" in value) {
      return getAttributeValueText(value.value);
    }

    if ("Value" in value) {
      return getAttributeValueText(value.Value);
    }

    if ("name" in value) {
      return getAttributeValueText(value.name);
    }

    if ("Name" in value) {
      return getAttributeValueText(value.Name);
    }
  }

  return "";
}

function getItemNameFromData(data: any) {
  const possibleNames = [
    data?.Name,
    data?.name,
    data?.ObjectType,
    data?.objectType,
    data?.Tag,
    data?.tag,
    data?.LongName,
    data?.longName,
    data?.GlobalId,
    data?.globalId,
  ];

  for (const possibleName of possibleNames) {
    const text = getAttributeValueText(possibleName).trim();

    if (text) {
      return text;
    }
  }

  return "";
}

async function showSelectedMepElementInfo() {
  const selectedNode = getFirstSelectedNode();

  if (!selectedNode) {
    selectedMepElementInfo.value = "Nenhum elemento classificado selecionado.";
    return;
  }

  const key = elementKey(selectedNode.modelId, selectedNode.localId);
  const definedElement = mepElements[key];

  if (!definedElement) {
    selectedMepElementInfo.value = "Elemento selecionado ainda sem classificação.";
    return;
  }

  const model = loadedModels.get(selectedNode.modelId);

  let elementName = "";

  if (model) {
    const [itemData] = await model.getItemsData([selectedNode.localId], {
      attributesDefault: true,
      relationsDefault: {
        attributes: true,
        relations: false,
      },
    });

    elementName = getItemNameFromData(itemData);
  }

  const typeLabel = getElementTypeLabel(definedElement.elementType);

  if (elementName) {
    selectedMepElementInfo.value = `${typeLabel}: ${elementName}`;
  } else {
    selectedMepElementInfo.value = `${typeLabel}: elemento #${selectedNode.localId}`;
  }
}

function getElementTypeLabel(elementType: MepElementType) {
  const labels: Record<MepElementType, string> = {
  pipe: "tubagem",
  isolationValve: "válvula de corte",
  normallyOpenValve: "válvula NA",
normallyClosedValve: "válvula NF",
  collector: "coletor",
  booster: "booster",
  reservoirWithResistance: "reservatório com resistência",
  reservoirWithoutResistance: "reservatório sem resistência",
};

  return labels[elementType];
}

function countMepElementsByType(elementType: MepElementType) {
  return Object.values(mepElements).filter(
    (element) => element.elementType === elementType,
  ).length;
}

function countDefinedMepElements() {
  return Object.keys(mepElements).length;
}

function getMepElementIdsByType(elementType: MepElementType) {
  const idsByModel = new Map<string, number[]>();

  for (const element of Object.values(mepElements)) {
    if (element.elementType !== elementType) {
      continue;
    }

    if (!idsByModel.has(element.modelId)) {
      idsByModel.set(element.modelId, []);
    }

    idsByModel.get(element.modelId)?.push(element.localId);
  }

  return idsByModel;
}

async function highlightMepElementsByType(elementType: MepElementType) {
  const idsByModel = getMepElementIdsByType(elementType);

  let highlightedCount = 0;

  for (const [modelId, ids] of idsByModel) {
    const model = loadedModels.get(modelId);

    if (!model || !ids.length) {
      continue;
    }

    await model.highlight(
      ids,
      createHighlight(
        mepElementHighlightColors[elementType],
        `mep-${elementType}`,
      ),
    );

    highlightedCount += ids.length;
  }

  await fragmentManager.core.update(true);

  flowMessage.value = highlightedCount
    ? `${highlightedCount} elemento(s) realçado(s) como ${getElementTypeLabel(elementType)}.`
    : `Não existem elementos definidos como ${getElementTypeLabel(elementType)}.`;
}

async function clearMepElementsHighlightByType(elementType: MepElementType) {
  const idsByModel = getMepElementIdsByType(elementType);

  let clearedCount = 0;

  for (const [modelId, ids] of idsByModel) {
    const model = loadedModels.get(modelId);

    if (!model || !ids.length) {
      continue;
    }

    await model.resetHighlight(ids);

    clearedCount += ids.length;
  }

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    await fragmentManager.core.update(true);
  }

  flowMessage.value = clearedCount
    ? `Realce removido de ${clearedCount} elemento(s) ${getElementTypeLabel(elementType)}.`
    : `Não havia elementos para limpar em ${getElementTypeLabel(elementType)}.`;
}

function isIsolationValve(modelId: string, localId: number) {
  const key = elementKey(modelId, localId);

  return (
    mepElements[key]?.elementType === "isolationValve" ||
    mepElements[key]?.elementType === "normallyOpenValve" ||
    mepElements[key]?.elementType === "normallyClosedValve"
  );
}

function updateBlockedCount() {
  blockedCount.value = [...blockedPipes.values()].reduce(
    (total, ids) => total + ids.size,
    0,
  );
}

async function setSelectedValvesState(state: "open" | "closed") {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro uma ou mais válvulas.";
    return;
  }

  let changedCount = 0;

  for (const [modelId, ids] of selectedItems) {
    let blockedSet = blockedPipes.get(modelId);

    if (!blockedSet) {
      blockedSet = new Set<number>();
      blockedPipes.set(modelId, blockedSet);
    }

    for (const localId of ids) {
      if (!isIsolationValve(modelId, localId)) {
        continue;
      }

      const key = elementKey(modelId, localId);

      mepElements[key] = {
        ...mepElements[key],
        modelId,
        localId,
        elementType: "isolationValve",
        circuitType: mepElements[key]?.circuitType ?? "unknown",
        state,
      };

      const valveNode: FlowNode = {
  modelId,
  localId,
};

const valveKey = nodeKey(valveNode);

if (state === "closed") {
  blockedSet.add(localId);

  const downstreamPipes = await findDownstreamPipesAfterValve(valveNode);

  for (const pipeNode of downstreamPipes) {
    let pipeBlockedSet = blockedPipes.get(pipeNode.modelId);

    if (!pipeBlockedSet) {
      pipeBlockedSet = new Set<number>();
      blockedPipes.set(pipeNode.modelId, pipeBlockedSet);
    }

    pipeBlockedSet.add(pipeNode.localId);
  }

  valveBlockedPipeLinks.set(valveKey, downstreamPipes);
} else {
  blockedSet.delete(localId);

  const linkedPipes = valveBlockedPipeLinks.get(valveKey) ?? [];

  for (const pipeNode of linkedPipes) {
    const linkedBlockedSet = blockedPipes.get(pipeNode.modelId);

    linkedBlockedSet?.delete(pipeNode.localId);

    if (linkedBlockedSet && linkedBlockedSet.size === 0) {
      blockedPipes.delete(pipeNode.modelId);
    }
  }

  valveBlockedPipeLinks.delete(valveKey);
}
      changedCount++;
    }

    if (blockedSet.size === 0) {
      blockedPipes.delete(modelId);
    }
  }

  updateBlockedCount();

  if (!changedCount) {
    flowMessage.value =
      "Nenhuma válvula de corte selecionada. Define primeiro o elemento como Válvula corte.";
    return;
  }

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  saveMepElementsToStorage();
  
  flowMessage.value =
    state === "closed"
      ? `${changedCount} válvula(s) fechada(s).`
      : `${changedCount} válvula(s) aberta(s).`;
}

async function closeSelectedValves() {
  await setSelectedValvesState("closed");
}

async function openSelectedValves() {
  await setSelectedValvesState("open");
}

async function hideSelectedFlowArrows() {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro um ou mais elementos para ocultar as setas.";
    return;
  }

  let hiddenCount = 0;

  for (const [modelId, ids] of selectedItems) {
    let hiddenSet = hiddenFlowArrowElements.get(modelId);

    if (!hiddenSet) {
      hiddenSet = new Set<number>();
      hiddenFlowArrowElements.set(modelId, hiddenSet);
    }

    for (const localId of ids) {
      hiddenSet.add(localId);
      hiddenCount++;
    }
  }

  saveHiddenFlowArrowsToStorage();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value = `${hiddenCount} elemento(s) sem setas de fluxo.`;
}

async function showSelectedFlowArrows() {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro um ou mais elementos para voltar a mostrar as setas.";
    return;
  }

  let shownCount = 0;

  for (const [modelId, ids] of selectedItems) {
    const hiddenSet = hiddenFlowArrowElements.get(modelId);

    if (!hiddenSet) {
      continue;
    }

    for (const localId of ids) {
      if (hiddenSet.delete(localId)) {
        shownCount++;
      }
    }

    if (hiddenSet.size === 0) {
      hiddenFlowArrowElements.delete(modelId);
    }
  }

  saveHiddenFlowArrowsToStorage();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value = shownCount
    ? `${shownCount} elemento(s) voltaram a mostrar setas.`
    : "Os elementos selecionados já mostravam setas.";
}

async function reverseSelectedPipesDirection() {
  if (!selectedCount.value) {
    flowMessage.value =
      "Seleciona primeiro um ou mais tubos para sincronizar o sentido.";
    return;
  }

  let changedCount = 0;

  for (const [modelId, ids] of selectedItems) {
    const syncedSet = getSyncedDirectionSet(modelId);

    for (const localId of ids) {
      if (syncedSet.has(localId)) {
        syncedSet.delete(localId);
      } else {
        syncedSet.add(localId);
      }

      toggleReversedPipeDirection(modelId, localId);

      changedCount++;
    }

    if (syncedSet.size === 0) {
      syncedPipeDirections.delete(modelId);
    }
  }

  saveSyncedPipeDirectionsToStorage();
  saveReversedDirectionsToStorage();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value =
    `${changedCount} tubo(s) sincronizado(s) com o sentido do caminho.`;
}

async function clearBlockedPipes() {
  blockedPipes.clear();
  valveBlockedPipeLinks.clear();

  for (const key of Object.keys(mepElements)) {
    const element = mepElements[key];

    if (element.elementType === "isolationValve") {
      mepElements[key] = {
        ...element,
        state: "open",
      };
    }
  }

  updateBlockedCount();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  saveMepElementsToStorage();
  
  flowMessage.value = "Bloqueios removidos e válvulas abertas.";
}

async function getPipeDirectionHints(node: FlowNode): Promise<PipeDirectionHints> {
  const hints: PipeDirectionHints = {};

  for (const connection of flowConnections) {
    if (isConnectionBlocked(connection)) {
      continue;
    }

    if (isSameNode(connection.to, node)) {
      hints.upstream = await getNodeCenter(connection.from);
    }

    if (isSameNode(connection.from, node)) {
      hints.downstream = await getNodeCenter(connection.to);
    }
  }

  return hints;
}

function isSameNode(a: FlowNode, b: FlowNode) {
  return a.modelId === b.modelId && a.localId === b.localId;
}

function getFirstSelectedNode(): FlowNode | null {
  for (const [modelId, ids] of selectedItems) {
    const [localId] = ids;
    if (Number.isFinite(localId)) return { modelId, localId };
  }

  return null;
}

function formatNodeLabel(node: FlowNode) {
  return `${node.modelId} #${node.localId}`;
}

function getCircuitLabel(circuit: PipeCircuit) {
  const labels: Record<PipeCircuit, string> = {
    supply1: "avanço 1",
    supply2: "avanço 2",
    supply3: "avanço 3",
    return1: "retorno 1",
    return2: "retorno 2",
    return3: "retorno 3",
  };

  return labels[circuit];
}

function getRoutesFromGroup(group: SavedRouteGroup) {
  return savedRoutes.filter((route) =>
    group.routeIds.includes(route.id),
  );
}

function getGroupForRoute(routeId: string) {
  const activeGroup = savedRouteGroups.find(
    (group) =>
      group.id === activeRouteGroupId.value &&
      group.routeIds.includes(routeId),
  );

  if (activeGroup) {
    return activeGroup;
  }

  return savedRouteGroups.find(
    (group) => group.routeIds.includes(routeId),
  ) ?? null;
}

function getRouteNamesFromGroup(group: SavedRouteGroup) {
  return getRoutesFromGroup(group).map((route) => route.name);
}

function getRouteDirectionAppearance(
  route: SavedRoute,
  nodeIndex: number,
): RouteDirectionAppearance {
  const previousNode = route.path[nodeIndex - 1] ?? null;
  const nextNode = route.path[nodeIndex + 1] ?? null;

  return {
    routeName: route.name,
    previousKey: previousNode ? nodeKey(previousNode) : null,
    nextKey: nextNode ? nodeKey(nextNode) : null,
  };
}

function directionAppearancesAreOpposite(
  first: RouteDirectionAppearance,
  second: RouteDirectionAppearance,
) {
  if (
    first.previousKey &&
    first.nextKey &&
    second.previousKey &&
    second.nextKey
  ) {
    return (
      first.previousKey === second.nextKey &&
      first.nextKey === second.previousKey
    );
  }

  if (first.nextKey && second.previousKey) {
    return first.nextKey === second.previousKey;
  }

  if (first.previousKey && second.nextKey) {
    return first.previousKey === second.nextKey;
  }

  return false;
}

function findSharedRouteDirectionConflicts(
  group: SavedRouteGroup,
): SharedRouteDirectionConflict[] {
  const routes = getRoutesFromGroup(group);
  const appearancesByNode = new Map<
    string,
    {
      node: FlowNode;
      appearances: RouteDirectionAppearance[];
    }
  >();

  for (const route of routes) {
    route.path.forEach((node, nodeIndex) => {
      const key = nodeKey(node);

      if (!appearancesByNode.has(key)) {
        appearancesByNode.set(key, {
          node,
          appearances: [],
        });
      }

      appearancesByNode.get(key)?.appearances.push(
        getRouteDirectionAppearance(route, nodeIndex),
      );
    });
  }

  const conflicts: SharedRouteDirectionConflict[] = [];

  for (const item of appearancesByNode.values()) {
    if (item.appearances.length < 2) {
      continue;
    }

    let hasConflict = false;

    for (let index = 0; index < item.appearances.length; index++) {
      for (
        let compareIndex = index + 1;
        compareIndex < item.appearances.length;
        compareIndex++
      ) {
        if (
          directionAppearancesAreOpposite(
            item.appearances[index],
            item.appearances[compareIndex],
          )
        ) {
          hasConflict = true;
        }
      }
    }

    if (hasConflict) {
      conflicts.push({
        node: item.node,
        routeNames: [
          ...new Set(
            item.appearances.map((appearance) => appearance.routeName),
          ),
        ],
      });
    }
  }

  return conflicts;
}

function routeContainsNode(route: SavedRoute, node: FlowNode) {
  return route.path.some((routeNode) => isSameNode(routeNode, node));
}

function isRouteHiddenByGroup(route: SavedRoute) {
  return savedRouteGroups.some(
    (group) => group.hidden && group.routeIds.includes(route.id),
  );
}

function isNodeHiddenBySavedRouteVisibility(node: FlowNode) {
  return savedRoutes.some(
    (route) =>
      (route.hidden || isRouteHiddenByGroup(route)) &&
      routeContainsNode(route, node),
  );
}

function getNextRouteNumberForCircuit(circuit: PipeCircuit) {
  return (
    savedRoutes.filter(
      (route) => route.temperature === circuit,
    ).length + 1
  );
}

function getNodeTemperature(node: FlowNode): PipeCircuit | null {
  const circuits: PipeCircuit[] = [
    "supply1",
    "supply2",
    "supply3",
    "return1",
    "return2",
    "return3",
  ];

  for (const circuit of circuits) {
    if (getAssignmentSet(circuit, node.modelId).has(node.localId)) {
      return circuit;
    }
  }

  return null;
}

async function getNodeCenter(node: FlowNode) {
  const model = loadedModels.get(node.modelId);
  if (!model) return null;

  const [box] = await model.getBoxes([node.localId]);
  if (!box) return null;

  const center = new THREE.Vector3();
  box.getCenter(center);
  return center;
}

function addConnectionParticles(start: any, end: any, temperature: PipeCircuit) {

  const direction = end.clone().sub(start);
  const length = Math.max(direction.length(), 0.1);
  direction.normalize();

  const radius = 0.055;
  const geometry = new THREE.ConeGeometry(radius * 1.3, radius * 2.8, 10);
  const material = circuitMaterials[temperature];
  const particleCount = Math.max(2, Math.round(length / 0.65));

  for (let i = 0; i < particleCount; i++) {
    const mesh = new THREE.Mesh(geometry, material);
    mesh.renderOrder = 30;
    mesh.quaternion.setFromUnitVectors(new THREE.Vector3(0, 1, 0), direction);
    mesh.position.copy(start);
    flowGroup.add(mesh);

    pipeParticles.push({
      mesh,
      start,
      end,
      offset: i / particleCount,
      length,
    });
  }
}

function addConnectionLine(start: any, end: any, temperature: PipeCircuit) {
  const geometry = new THREE.BufferGeometry().setFromPoints([start, end]);

  const material = new THREE.LineBasicMaterial({
  color: {
    supply1: 0xff0000,
    supply2: 0xff5252,
    supply3: 0xff8a80,
    return1: 0xff8c00,
    return2: 0xffa726,
    return3: 0xffc107,
  }[temperature],
    transparent: true,
    opacity: 0.35,
    depthTest: false,
  });
  const line = new THREE.Line(geometry, material);
  line.renderOrder = 15;
  flowGroup.add(line);
  staticFlowObjects.push({ object: line });
}

function classifyPipe(data: FRAGS.ItemData | undefined): PipeCircuit {
  const text = flattenItemText(data).toLowerCase();

  const hotTerms = [
    "quente",
    "hot",
    "aqs",
    "acs",
    "dhw",
    "heating",
  ];

  const hasHot = hotTerms.some((term) => text.includes(term));

  return hasHot ? "supply1" : "return1";
}

function flattenItemText(value: unknown): string {
  if (value === null || value === undefined) return "";
  if (Array.isArray(value)) return value.map(flattenItemText).join(" ");
  if (typeof value === "object") {
    return Object.entries(value)
      .map(([key, nested]) => `${key} ${flattenItemText(nested)}`)
      .join(" ");
  }
  return String(value);
}

function createHighlight(color: number, customId: string): FRAGS.MaterialDefinition {
  return {
    color: new THREE.Color(color),
    renderedFaces: FRAGS.RenderedFaces.TWO,
    opacity: 0.9,
    transparent: true,
    customId,
  };
}

function addPipeParticles(
  box: any,
  temperature: PipeCircuit,
  hints: PipeDirectionHints = {},
  node?: FlowNode,
)

 {
  const size = new THREE.Vector3();
  const center = new THREE.Vector3();

  box.getSize(size);
  box.getCenter(center);

  const axis = getLongestAxis(size);
  const length = Math.max(size.getComponent(axis), 0.1);

  const endpointA = center.clone();
  const endpointB = center.clone();

  endpointA.setComponent(axis, center.getComponent(axis) - length / 2);
  endpointB.setComponent(axis, center.getComponent(axis) + length / 2);

  let { start, end } = choosePipeDirection(endpointA, endpointB, hints);

if (node && isPipeDirectionReversed(node.modelId, node.localId)) {
  const originalStart = start;
  start = end;
  end = originalStart;
}

const direction = end.clone().sub(start).normalize();

  const radius = 0.04;
  const geometry = new THREE.ConeGeometry(radius * 1.2, radius * 2.5, 8);

  const material = circuitMaterials[temperature];

  const particleCount = Math.max(1, Math.round(length / 0.4));

  for (let i = 0; i < particleCount; i++) {
    const mesh = new THREE.Mesh(geometry, material);
    mesh.renderOrder = 20;

    mesh.quaternion.setFromUnitVectors(
      new THREE.Vector3(0, 1, 0),
      direction
    );

    mesh.position.copy(start);

    flowGroup.add(mesh);

    pipeParticles.push({
      mesh,
      start,
      end,
      offset: i / particleCount,
      length,
    });
  }
}

function choosePipeDirection(endpointA: any, endpointB: any, hints: PipeDirectionHints) {
  let start = endpointA;
  let end = endpointB;

  if (hints.upstream && hints.downstream) {
    start = closestEndpoint(endpointA, endpointB, hints.upstream);
    end = closestEndpoint(endpointA, endpointB, hints.downstream);

    if (start.distanceTo(end) < 0.001) {
      end = start === endpointA ? endpointB : endpointA;
    }

    return { start, end };
  }

  if (hints.upstream) {
    start = closestEndpoint(endpointA, endpointB, hints.upstream);
    end = start === endpointA ? endpointB : endpointA;
    return { start, end };
  }

  if (hints.downstream) {
    end = closestEndpoint(endpointA, endpointB, hints.downstream);
    start = end === endpointA ? endpointB : endpointA;
  }

  return { start, end };
}

function closestEndpoint(endpointA: any, endpointB: any, point: any) {
  return endpointA.distanceTo(point) <= endpointB.distanceTo(point)
    ? endpointA
    : endpointB;
}

function getLongestAxis(size: any) {
  if (size.x >= size.y && size.x >= size.z) return 0;
  if (size.y >= size.x && size.y >= size.z) return 1;
  return 2;
}

function animateFlow() {
  const clockStart = performance.now();

  const FLOW_METERS_PER_SECOND = 1.0;

  const tick = () => {
    if (isFlowing.value && pipeParticles.length) {

      const elapsed =
        ((performance.now() - clockStart) / 1000) *
        flowSpeed.value;

      for (const particle of pipeParticles) {

        const progress =
          ((elapsed * FLOW_METERS_PER_SECOND) / particle.length +
            particle.offset) % 1;

        particle.mesh.position.lerpVectors(
          particle.start,
          particle.end,
          progress
        );
      }
    }

    animationFrame = requestAnimationFrame(tick);
  };

  tick();
}

function toggleElementPanelMinimized() {
  isElementPanelMinimized.value = !isElementPanelMinimized.value;
}

function toggleFlowControlsPanelMinimized() {
  isFlowControlsPanelMinimized.value = !isFlowControlsPanelMinimized.value;
}

function toggleSimulationControlPanelMinimized() {
  isSimulationControlPanelMinimized.value =
    !isSimulationControlPanelMinimized.value;
}

function getAllAssignedPipeNodes() {
  const nodes: FlowNode[] = [];

  for (const assignmentMap of Object.values(manualAssignments)) {
    for (const [modelId, ids] of assignmentMap) {
      for (const localId of ids) {
        nodes.push({
          modelId,
          localId,
        });
      }
    }
  }

  return nodes;
}

async function findClosestAssignedPipeToNode(node: FlowNode) {
  const nodeCenter = await getNodeCenter(node);

  if (!nodeCenter) {
    return null;
  }

  let closestNode: FlowNode | null = null;
  let closestDistance = Number.POSITIVE_INFINITY;

  for (const candidate of getAllAssignedPipeNodes()) {
    if (isSameNode(candidate, node)) {
      continue;
    }

    const candidateCenter = await getNodeCenter(candidate);

    if (!candidateCenter) {
      continue;
    }

    const distance = nodeCenter.distanceTo(candidateCenter);

    if (distance < closestDistance) {
      closestDistance = distance;
      closestNode = candidate;
    }
  }

  return closestNode;
}

async function findDownstreamPipesAfterValve(valveNode: FlowNode) {
  const valveCenter = await getNodeCenter(valveNode);

  if (!valveCenter || !flowConnections.length) {
    return [];
  }

  let closestConnectionIndex = -1;
  let closestDistance = Number.POSITIVE_INFINITY;

  for (let index = 0; index < flowConnections.length; index++) {
    const connection = flowConnections[index];

    const fromCenter = await getNodeCenter(connection.from);
    const toCenter = await getNodeCenter(connection.to);

    if (!fromCenter || !toCenter) {
      continue;
    }

    const middlePoint = fromCenter.clone().add(toCenter).multiplyScalar(0.5);
    const distance = valveCenter.distanceTo(middlePoint);

    if (distance < closestDistance) {
      closestDistance = distance;
      closestConnectionIndex = index;
    }
  }

  if (closestConnectionIndex === -1) {
    return [];
  }

  const downstreamNodes: FlowNode[] = [];
  const firstConnection = flowConnections[closestConnectionIndex];

  downstreamNodes.push(firstConnection.to);

  let expectedFrom = firstConnection.to;

  for (
    let index = closestConnectionIndex + 1;
    index < flowConnections.length;
    index++
  ) {
    const connection = flowConnections[index];

    if (!isSameNode(connection.from, expectedFrom)) {
      break;
    }

    downstreamNodes.push(connection.to);
    expectedFrom = connection.to;
  }

  return downstreamNodes;
}

function isPipeBlocked(modelId: string, localId: number) {
  return blockedPipes.get(modelId)?.has(localId) ?? false;
}

function isConnectionBlocked(connection: FlowConnection) {
  return (
    isPipeBlocked(connection.from.modelId, connection.from.localId) ||
    isPipeBlocked(connection.to.modelId, connection.to.localId)
  );
}

function getBlockedNodesInCurrentPath() {
  const blockedNodes: FlowNode[] = [];

  if (flowConnections.length) {
    for (const connection of flowConnections) {
      if (isPipeBlocked(connection.from.modelId, connection.from.localId)) {
        blockedNodes.push(connection.from);
      }

      if (isPipeBlocked(connection.to.modelId, connection.to.localId)) {
        blockedNodes.push(connection.to);
      }
    }
  } else {
    for (const [modelId, ids] of blockedPipes) {
      for (const localId of ids) {
        blockedNodes.push({ modelId, localId });
      }
    }
  }

  return blockedNodes;
}

function hasDefinedPumpOrHeatPump() {
  return Object.values(mepElements).some(
    (element) => element.elementType === "booster",
  );
}

async function startCentralSimulation() {
  if (!countAssignments()) {
    flowMessage.value =
  "Não é possível simular: marca primeiro tubos num circuito.";
    return;
  }

  const blockedNodes = getBlockedNodesInCurrentPath();

if (blockedNodes.length) {
  flowMessage.value =
    "Simulação iniciada com válvula(s)/tubo(s) bloqueado(s). O fluxo não passa nesses elementos.";
}

  await rebuildManualFlowLayer();

  if (!pipeParticles.length) {
    flowMessage.value =
      "Não é possível simular: não existem setas de fluxo criadas.";
    isCentralSimulationRunning.value = false;
    isFlowing.value = false;
    return;
  }

  isFlowManuallyPaused.value = false;
isCentralSimulationRunning.value = true;
isFlowing.value = true;

  if (!hasDefinedPumpOrHeatPump()) {
    flowMessage.value =
      "Simulação iniciada. Aviso: ainda não foi definido nenhum booster.";
    return;
  }

  flowMessage.value = "Simulação da central iniciada.";
}

function stopCentralSimulation() {
  isCentralSimulationRunning.value = false;
  isFlowing.value = false;
  isFlowManuallyPaused.value = true;
  flowMessage.value = "Simulação da central parada.";
}

async function toggleCentralSimulation() {
  if (isCentralSimulationRunning.value) {
    stopCentralSimulation();
    return;
  }

  await startCentralSimulation();
}


async function blockSelectedPipes() {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro um ou mais elementos para bloquear.";
    return;
  }

  for (const [modelId, ids] of selectedItems) {
    let blockedSet = blockedPipes.get(modelId);

    if (!blockedSet) {
      blockedSet = new Set<number>();
      blockedPipes.set(modelId, blockedSet);
    }

    for (const localId of ids) {
      blockedSet.add(localId);
    }
  }

  blockedCount.value = [...blockedPipes.values()].reduce(
    (total, ids) => total + ids.size,
    0,
  );

  flowMessage.value = `${selectedCount.value} elemento(s) bloqueado(s).`;
  await rebuildManualFlowLayer();
}

function toggleFlow() {
  if (!pipeParticles.length) {
    flowMessage.value = "Marca pelo menos um tubo antes de iniciar a animação.";
    return;
  }

  if (isFlowing.value) {
    isFlowing.value = false;
    isManualFlowAnimationRunning.value = false;
    isFlowManuallyPaused.value = true;
    flowMessage.value = "Animação pausada.";
    return;
  }

  isManualFlowAnimationRunning.value = true;
  isFlowManuallyPaused.value = false;
  isFlowing.value = true;
  flowMessage.value = "Animação iniciada.";
}

function clearFlowVisuals(keepFlowState = false) {
  for (const particle of pipeParticles) {
    flowGroup.remove(particle.mesh);

    particle.mesh.geometry.dispose();
  }

  for (const visual of staticFlowObjects) {
    flowGroup.remove(visual.object);
    visual.object.geometry?.dispose?.();
    visual.object.material?.dispose?.();
  }

  pipeParticles.length = 0;
  staticFlowObjects.length = 0;

  if (!keepFlowState) {
    isFlowing.value = false;
  }
}

function clearFlowLayer() {
  clearFlowVisuals();

  pipeStats.supply = 0;
  pipeStats.return = 0;

  pipeStats.supply1 = 0;
  pipeStats.supply2 = 0;
  pipeStats.supply3 = 0;

  pipeStats.return1 = 0;
  pipeStats.return2 = 0;
  pipeStats.return3 = 0;

  pipeStats.total = 0;
}

function replaceSelection(modelIdMap: unknown) {
  selectedItems.clear();

  for (const [modelId, ids] of normalizeModelIdMap(modelIdMap)) {
    if (ids.size) selectedItems.set(modelId, ids);
  }

  selectedCount.value = [...selectedItems.values()].reduce(
    (total, ids) => total + ids.size,
    0,
  );
}

function normalizeModelIdMap(modelIdMap: unknown): SelectionMap {
  const result: SelectionMap = new Map();

  if (modelIdMap instanceof Map) {
    for (const [modelId, value] of modelIdMap) {
      result.set(String(modelId), new Set(toNumberArray(value)));
    }
    return result;
  }

  if (modelIdMap && typeof modelIdMap === "object") {
    for (const [modelId, value] of Object.entries(modelIdMap)) {
      result.set(modelId, new Set(toNumberArray(value)));
    }
  }

  return result;
}

function toNumberArray(value: unknown): number[] {
  if (value instanceof Set) return [...value].map(Number).filter(Number.isFinite);
  if (Array.isArray(value)) return value.map(Number).filter(Number.isFinite);
  if (value && typeof value === "object") {
    return Object.values(value).flatMap(toNumberArray);
  }
  const number = Number(value);
  return Number.isFinite(number) ? [number] : [];
}

function getSyncedDirectionSet(modelId: string) {
  let ids = syncedPipeDirections.get(modelId);

  if (!ids) {
    ids = new Set<number>();
    syncedPipeDirections.set(modelId, ids);
  }

  return ids;
}

function isPipeDirectionSynced(modelId: string, localId: number) {
  return syncedPipeDirections.get(modelId)?.has(localId) ?? false;
}

function toggleReversedPipeDirection(modelId: string, localId: number) {
  const reversedSet = getReversedDirectionSet(modelId);

  if (reversedSet.has(localId)) {
    reversedSet.delete(localId);
  } else {
    reversedSet.add(localId);
  }

  if (reversedSet.size === 0) {
    reversedPipeDirections.delete(modelId);
  }
}

function toggleSyncedPipesForPath(path: FlowNode[]) {
  for (const node of path) {
    if (!isPipeDirectionSynced(node.modelId, node.localId)) {
      continue;
    }

    toggleReversedPipeDirection(node.modelId, node.localId);
  }

  saveReversedDirectionsToStorage();
}

function getReversedDirectionSet(modelId: string) {
  let ids = reversedPipeDirections.get(modelId);

  if (!ids) {
    ids = new Set<number>();
    reversedPipeDirections.set(modelId, ids);
  }

  return ids;
}

function isFlowArrowHidden(modelId: string, localId: number) {
  return hiddenFlowArrowElements.get(modelId)?.has(localId) ?? false;
}

function isPipeDirectionReversed(modelId: string, localId: number) {
  if (reversedPipeDirections.get(modelId)?.has(localId)) {
    return true;
  }

  for (const ids of reversedPipeDirections.values()) {
    if (ids.has(localId)) {
      return true;
    }
  }

  return false;
}

function getAssignmentSet(circuit: PipeCircuit, modelId: string) {
  let ids = manualAssignments[circuit].get(modelId);
  if (!ids) {
    ids = new Set<number>();
    manualAssignments[circuit].set(modelId, ids);
  }
  return ids;
}

function updateManualStats() {
  pipeStats.supply1 = countAssignmentType("supply1");
  pipeStats.supply2 = countAssignmentType("supply2");
  pipeStats.supply3 = countAssignmentType("supply3");

  pipeStats.return1 = countAssignmentType("return1");
  pipeStats.return2 = countAssignmentType("return2");
  pipeStats.return3 = countAssignmentType("return3");

  pipeStats.supply =
    pipeStats.supply1 +
    pipeStats.supply2 +
    pipeStats.supply3;

  pipeStats.return =
    pipeStats.return1 +
    pipeStats.return2 +
    pipeStats.return3;

  pipeStats.total =
    pipeStats.supply +
    pipeStats.return;
}

function countAssignmentType(circuit: PipeCircuit) {
  return [...manualAssignments[circuit].values()].reduce(
    (total, ids) => total + ids.size,
    0,
  );
}

function countAssignments() {
  return (
    countAssignmentType("supply1") +
    countAssignmentType("supply2") +
    countAssignmentType("supply3") +
    countAssignmentType("return1") +
    countAssignmentType("return2") +
    countAssignmentType("return3")
  );
}

function chunk<T>(items: T[], size: number) {
  const result: T[][] = [];
  for (let index = 0; index < items.length; index += size) {
    result.push(items.slice(index, index + size));
  }
  return result;
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 42, 58, 0.6);
  backdrop-filter: blur(6px);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: all;
}

.loading-box {
  text-align: center;
  color: white;
  font-family: sans-serif;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 12px;
}

.loading-text {
  font-size: 1rem;
  font-weight: 500;
}

.corner-logo {
  position: fixed;
  bottom: 32px;
  right: 32px;
  border-radius: 5px;
  height: 64px;
  width: 64px;
  z-index: 1001;
  opacity: 1;
}

.corner-logo img {
  height: 100%;
  border-radius: 5px;
  transition: all 0.3s ease;
  position: absolute;
  width: auto;
}

.app-logo {
  opacity: 1;
}

.control-panels {
  position: fixed;
  right: 24px;
  top: 24px;
  z-index: 1000;
  width: min(390px, calc(100vw - 48px));
  max-height: calc(100vh - 48px);
  overflow-y: auto;
  display: grid;
  gap: 14px;
}

.flow-panel {
  width: 100%;
  padding: 16px;
  border: 1px solid rgba(255, 255, 255, 0.22);
  border-radius: 8px;
  background: rgba(13, 22, 28, 0.88);
  color: #f7fbff;
  box-shadow: 0 16px 38px rgba(0, 0, 0, 0.24);
  backdrop-filter: blur(10px);
}

.flow-panel__header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
}

.flow-panel__toggle {
  display: inline-grid;
  width: 28px;
  height: 28px;
  place-items: center;
  border: 0;
  border-radius: 999px;
  background: #f7fbff;
  color: #111820;
  cursor: pointer;
  font-size: 1.1rem;
  font-weight: 900;
  line-height: 1;
}

.flow-panel__toggle:hover {
  background: #d9f0ff;
}

.flow-panel--minimized {
  width: min(300px, calc(100vw - 48px));
  max-height: none;
  overflow: hidden;
}

.flow-panel--minimized .flow-panel__content {
  display: none;
}

.flow-panel__eyebrow {
  margin: 0 0 2px;
  color: #8fd3ff;
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
}

.flow-panel h2 {
  margin: 0;
  font-size: 1.05rem;
  font-weight: 700;
}

.flow-status {
  display: inline-grid;
  min-width: 44px;
  min-height: 28px;
  place-items: center;
  border-radius: 999px;
  background: #3d474d;
  color: #cbd5dc;
  font-size: 0.75rem;
  font-weight: 800;
}

.flow-status--on {
  background: #e53935;
  color: white;
}


.flow-section-title {
  margin-top: 16px;
  padding-top: 12px;
  border-top: 1px solid rgba(255, 255, 255, 0.18);
  color: #8fd3ff;
  font-size: 0.78rem;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.flow-actions {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 8px;
  margin-top: 14px;
}

.flow-actions--single {
  grid-template-columns: 1fr;
}

.flow-actions button {
  min-width: 0;
  min-height: 36px;
  border: 0;
  border-radius: 6px;
  background: #f7fbff;
  color: #111820;
  cursor: pointer;
  font-weight: 700;
  white-space: normal;
  overflow-wrap: break-word;
}

.flow-actions button:hover {
  background: #d9f0ff;
}

.flow-button--danger {
  background: #ffe3e3 !important;
  color: #7a1010 !important;
}

.flow-button--primary {
  min-height: 44px !important;
  font-size: 0.9rem;
  font-weight: 800;
}

.flow-button--danger:hover {
  background: #ffc9c9 !important;
}

.flow-slider {
  display: grid;
  gap: 8px;
  margin-top: 14px;
  font-size: 0.85rem;
  font-weight: 700;
}

.flow-slider input {
  width: 100%;
}

.flow-stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 8px;
  margin: 14px 0 0;
}

.flow-stats div {
  min-width: 0;
  padding: 10px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
}

.flow-stats dt {
  color: #b8c9d3;
  font-size: 0.72rem;
  font-weight: 700;
}

.selected-mep-info {
  margin-top: 10px;
  padding: 10px;
  border-radius: 6px;
  background: rgba(143, 211, 255, 0.14);
  color: #f7fbff;
  font-size: 0.82rem;
  font-weight: 800;
  line-height: 1.35;
}

.element-highlight-list {
  display: grid;
  gap: 8px;
  margin-top: 12px;
}

.element-highlight-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding: 8px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
  font-size: 0.78rem;
  font-weight: 700;
}

.element-highlight-item button {
  border: 0;
  border-radius: 4px;
  padding: 5px 10px;
  cursor: pointer;
  font-size: 0.72rem;
  font-weight: 800;
  background: #f7fbff;
  color: #111820;
}

.element-highlight-item button:hover {
  background: #d9f0ff;
}

.flow-note {
  margin: 12px 0 0;
  color: #dbe9f1;
  font-size: 0.82rem;
  line-height: 1.4;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.saved-routes {
  margin-top: 14px;
}

.saved-route-item {
  margin-top: 6px;
  padding: 8px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);

  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 8px;

  font-size: 0.8rem;
}

.saved-route-item button {
  border: 0;
  border-radius: 4px;
  padding: 4px 6px;
  cursor: pointer;
  font-size: 0.72rem;
}

.saved-route-item--group {
  align-items: flex-start;
}

.saved-route-group-info {
  display: grid;
  gap: 4px;
}

.saved-route-group-count {
  margin: 0;
  color: #b8c9d3;
  font-size: 0.72rem;
  font-weight: 700;
}

.saved-route-group-list {
  margin: 4px 0 0;
  padding-left: 16px;
  color: #dbe9f1;
  font-size: 0.72rem;
  line-height: 1.35;
}

.saved-route-select {
  display: flex;
  align-items: center;
  gap: 6px;
}

.saved-route-select input {
  cursor: pointer;
}

.saved-route-item > div:last-child {
  display: flex;
  gap: 4px;
  flex-wrap: wrap;
  justify-content: flex-end;
}

.manual-route-mode-button {
  min-height: 44px;
  border: 0;
  border-radius: 6px;
  background: #f7fbff;
  color: #111820;
  cursor: pointer;
  font-size: 0.88rem;
  font-weight: 800;
}

.manual-route-mode-button:hover {
  background: #d9f0ff;
}

.manual-route-mode-button--active {
  background: #8fd3ff !important;
  color: #07131a !important;
  box-shadow: 0 0 0 2px rgba(143, 211, 255, 0.35);
}

.manual-route-status {
  margin: 8px 0 0;
  padding: 8px 10px;
  border-radius: 6px;
  background: rgba(143, 211, 255, 0.18);
  color: #8fd3ff;
  font-size: 0.82rem;
  font-weight: 900;
  line-height: 1.35;
}

.route-warning-message {
  margin: 12px 0 0;
  padding: 10px;
  border-radius: 6px;
  background: rgba(255, 193, 7, 0.18);
  color: #ffc107;
  font-size: 0.82rem;
  font-weight: 900;
  line-height: 1.35;
  border: 1px solid rgba(255, 193, 7, 0.35);
}

@media (max-width: 820px) {
  .control-panels {
    top: auto;
    right: 12px;
    bottom: 12px;
    width: calc(100vw - 24px);
    max-height: calc(100vh - 24px);
  }

  .corner-logo {
    display: none;
  }
}
</style>
