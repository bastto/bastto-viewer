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
    <button
  type="button"
  class="ifc-panel-side-toggle"
  :class="{ 'ifc-panel-side-toggle--collapsed': isIfcPanelCollapsed }"
  @click="toggleIfcPanelCollapsed"
>
  {{ isIfcPanelCollapsed ? '›' : '‹' }}
</button>
    <div
  v-if="cycleCircuitDefinitions.length"
  class="global-color-legend"
  :class="{ 'global-color-legend--ifc-collapsed': isIfcPanelCollapsed }"
>
  <div class="global-color-legend__title">
    Legenda de cores
  </div>

  <div class="global-color-legend__list">
  <div
    v-for="cycleGroup in getCycleCircuitLegendGroups()"
    :key="`legend-cycle-group-${cycleGroup.cycleNumber}`"
    class="global-color-legend__group"
  >
    <div class="global-color-legend__cycle-title">
      {{ cycleGroup.cycleName }}
    </div>

    <div
      v-for="circuit in cycleGroup.circuits"
      :key="`global-legend-${circuit.key}`"
      class="global-color-legend__item"
    >
      
      <span
  class="cycle-color-dot"
  :style="{ backgroundColor: circuit.color }"
></span>

      <span class="global-color-legend__text">
        {{ circuit.name }}
      </span>
    </div>
  </div>
</div>
</div>

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
    Apagar definição selecionada
  </button>

  <button
    type="button"
    class="flow-button--danger"
    @click="deleteAllElementDefinitions"
  >
    Apagar todas as definições
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
  Configuração da central
</div>

<label class="flow-cycle-config">
  <span>Número de ciclos de água</span>

  <input
    v-model.number="pendingWaterCycleCount"
    type="number"
    min="1"
    max="12"
  />
</label>

<div class="flow-actions flow-actions--single">
  <button type="button" @click="applyWaterCycleCount">
    Aplicar ciclos
  </button>
</div>

<div class="flow-section-title flow-section-title--button">
  <span>Renomear ciclos</span>
  <button
    type="button"
    class="section-collapse-button"
    @click="toggleCycleNamesPanel"
  >
    {{ isCycleNamesPanelOpen ? '−' : '+' }}
  </button>
</div>

<div v-if="isCycleNamesPanelOpen" class="cycle-name-list">
  <label
    v-for="cycleNumber in waterCycleCount"
    :key="`cycle-name-${cycleNumber}`"
    class="cycle-name-item"
  >
    <span>Ciclo {{ cycleNumber }}</span>
    <input
      v-model="cycleNames[cycleNumber]"
      type="text"
      :placeholder="`Ex: AQS, Aquecimento, Água fria...`"
    />
  </label>

  <div class="flow-actions flow-actions--single">
    <button type="button" @click="saveCycleNames">
      Guardar nomes dos ciclos
    </button>
  </div>
</div>

<div class="flow-section-title">
  1. Escolher ciclo e caminho
</div>


<label class="flow-cycle-config">
  <span>Ciclo onde vou trabalhar</span>
  <select
    v-model.number="activeCycleNumber"
    @change="selectDefaultCircuitForActiveCycle"
  >
    <option
      v-for="cycleNumber in waterCycleCount"
      :key="`active-cycle-${cycleNumber}`"
      :value="cycleNumber"
    >
      {{ getCycleDisplayName(cycleNumber) }}
    </option>
  </select>
</label>

<p
  v-if="!getActiveCycleCircuitDefinitions().length"
  class="connection-note workflow-help-note"
>
  Este ciclo ainda não tem caminhos. Cria um caminho em baixo para começar.
</p>

<div class="flow-section-title flow-section-title--button">
  <span>Criar ou editar caminhos deste ciclo</span>
  <button
    type="button"
    class="section-collapse-button"
    @click="toggleCycleCircuitPanel"
  >
    {{ isCycleCircuitPanelOpen ? '−' : '+' }}
  </button>
</div>

<div v-if="isCycleCircuitPanelOpen" class="cycle-circuit-panel">
  <label class="flow-cycle-config">
    <span>Tipo de caminho</span>
    <select
  v-model="pendingCycleCircuitKind"
  @change="updatePendingCycleCircuitColorFromKind"
>
  <option value="hotSupply">Ida quente</option>
<option value="coldSupply">Ida fria</option>
<option value="hotReturn">Retorno quente</option>
<option value="coldReturn">Retorno frio</option>
<option value="extra">Extra</option>
</select>
  </label>

  <label class="flow-cycle-config">
    <span>Nome do caminho</span>
    <input
  v-model="pendingCycleCircuitName"
  type="text"
  placeholder="Ex: Nome do caminho extra"
/>
  </label>

  <label class="flow-cycle-config">
    <span>Cor</span>
    <input
      v-model="pendingCycleCircuitColor"
      type="color"
    />
  </label>

  <div class="flow-actions flow-actions--single">
    <button type="button" @click="createCycleCircuitDefinition">
      Criar e selecionar caminho
    </button>

    <button type="button" @click="saveCycleCircuitDefinitionChanges">
      Guardar alterações
    </button>
  </div>

  <p
  v-if="!getActiveCycleCircuitDefinitions().length"
  class="connection-note workflow-help-note"
>
  Este ciclo ainda não tem caminhos. Cria um caminho para começar.
</p>

  <div class="cycle-circuit-list">
    <div
      v-for="circuit in getActiveCycleCircuitDefinitions()"
      :key="circuit.key"
      :class="[
  'cycle-circuit-item',
  'cycle-circuit-item--simple',
  circuit.locked ? 'cycle-circuit-item--locked' : ''
]"
    >
      <span
        class="cycle-color-dot"
        :style="{ backgroundColor: circuit.color }"
      ></span>

      <div class="cycle-circuit-name-wrapper">
  <span
    v-if="circuit.locked"
    class="cycle-circuit-lock-icon"
    title="Caminho protegido"
  >
    🔒
  </span>

  <input
    v-model="circuit.name"
    type="text"
    :disabled="circuit.locked"
  />
</div>

      <button
  type="button"
  :class="[
    'cycle-circuit-use-button',
    selectedCycleCircuitKey === circuit.key
      ? 'cycle-circuit-use-button--active'
      : ''
  ]"
  @click="selectedCycleCircuitKey = circuit.key"
>
  {{ selectedCycleCircuitKey === circuit.key ? 'Selecionado' : 'Usar' }}
</button>

<button
  type="button"
  :class="[
    'cycle-circuit-lock-button',
    circuit.locked ? 'cycle-circuit-lock-button--active' : ''
  ]"
  @click="toggleCycleCircuitLock(circuit.key)"
>
  {{ circuit.locked ? 'Desproteger' : 'Proteger' }}
</button>

     <button
  v-if="isCycleCircuitColorChanged(circuit) && !circuit.locked"
  type="button"
  @click="resetCycleCircuitColor(circuit.key)"
>
  Repor cor
</button>

      <button
  v-if="!circuit.locked"
  type="button"
  class="flow-button--danger"
  @click="deleteCycleCircuitDefinition(circuit.key)"
>
  Apagar
</button>
    </div>
  </div>
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
  2. Criar percurso
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
  Modo manual ativo: seleciona os tubos pela ordem do percurso.
</p>

<p class="connection-note">
  Caminho manual: {{ manualRouteNodes.length }} tubo(s)
</p>

<div
  v-if="isManualRouteRecording && manualRouteNodes.length > 0"
  class="flow-actions flow-actions--single"
>
  <button type="button" @click.stop.prevent="removeLastManualRouteNode">
  Remover último tubo
</button>
</div>

<p v-if="!isManualRouteRecording" class="connection-note workflow-help-note">
  Para criar automaticamente: seleciona o tubo inicial, define início, seleciona o tubo final, define fim e cria o percurso.
</p>

<div v-if="!isManualRouteRecording" class="flow-actions flow-actions--secondary">
  <button type="button" @click="setRouteStart">
    1. Definir início
  </button>
  <button type="button" @click="setRouteEnd">
    2. Definir fim
  </button>
</div>

<div v-if="!isManualRouteRecording" class="flow-actions flow-actions--single">
  <button type="button" @click="createAutoRouteForSelectedCycleCircuit">
    3. Criar caminho automático
  </button>
</div>

<div
  v-if="isManualRouteRecording && manualRouteNodes.length >= 2"
  class="flow-section-title"
>
  Criar percurso manual
</div>

  <div
  v-if="isManualRouteRecording && manualRouteNodes.length >= 2"
  class="flow-actions flow-actions--single"
>
  <button type="button" @click="createManualRouteForSelectedCycleCircuit">
    Criar caminho manual
  </button>
</div>

    <div
  v-if="currentRouteConnections.length || manualRouteNodes.length"
  class="flow-section-title"
>
  3. Guardar ou descartar
</div>

<div
  v-if="currentRouteConnections.length || manualRouteNodes.length"
  class="flow-actions flow-actions--single"
>
  <button
    type="button"
    class="flow-button--primary"
    @click="saveCurrentRoute"
  >
    Guardar caminho
  </button>

  <button
    type="button"
    class="flow-button--danger"
    @click="discardCurrentRoute"
  >
    Descartar caminho atual
  </button>
</div>

<p v-if="discardRouteMessage" class="discard-route-message">
  {{ discardRouteMessage }}
</p>

 <div v-if="hasLoadedModel && savedRoutes.length" class="saved-routes">
  <div class="saved-routes-header">
    <span>
      Caminhos guardados: {{ savedRoutes.length }}
    </span>

    <button
      type="button"
      class="section-collapse-button"
      @click="toggleSavedRoutesPanel"
    >
      {{ isSavedRoutesPanelOpen ? '−' : '+' }}
    </button>
  </div>

  <div v-if="isSavedRoutesPanelOpen" class="saved-routes-list">
    <div
      v-for="route in savedRoutes"
      :key="route.id"
      class="saved-route-item"
      :class="{
  'saved-route-item--locked': route.locked,  
  'saved-route-item--highlighted': highlightedSavedRouteId === route.id,
  'saved-route-item--blocked': isSavedRouteBlocked(route)
}"
    >
      <div class="saved-route-select saved-route-select--details">
  <span class="saved-route-text">
          <strong>
            <span v-if="route.locked" class="route-lock-icon">🔒</span>
            {{ route.name }}
          </strong>

          <small>
            {{ getRouteCircuitDisplayLabel(route) }} ·
{{ getRoutePipeCount(route) }} tubo(s) ·
{{ getRouteVisibilityLabel(route) }} ·
{{ getRouteProtectionLabel(route) }} ·
{{ getRouteBlockedLabel(route) }}
<span v-if="isSavedRouteBlocked(route)">
  · {{ getValveLabelForBlockedRoute(route) }}
</span>
          </small>
        </span>
      </div>

      <div>
        <button type="button" @click="applySavedRoute(route)">
          Aplicar
        </button>

        <button type="button" @click="toggleSavedRouteHighlight(route)">
  {{ highlightedSavedRouteId === route.id ? 'Limpar realce' : 'Realçar' }}
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

        <button
          v-if="!route.locked"
          type="button"
          @click="reverseSavedRoute(route.id)"
        >
          Inverter
        </button>

        <button
          v-if="!route.locked"
          type="button"
          @click="renameSavedRoute(route.id)"
        >
          Renomear
        </button>

        <button type="button" @click="toggleSavedRouteProtection(route.id)">
          {{ route.locked ? 'Desproteger' : 'Proteger' }}
        </button>

        <button
          v-if="!route.locked"
          type="button"
          @click="deleteSavedRoute(route.id)"
        >
          Apagar
        </button>
      </div>
    </div>
  </div>
</div>

    <p class="connection-note">Inicio: {{ routeStartLabel }}</p>
    <p class="connection-note">Fim: {{ routeEndLabel }}</p>

    <dl class="flow-stats">
  <template
  v-for="circuit in getActiveCycleCircuitDefinitions()"
  :key="`stats-active-circuit-${circuit.key}`"
>
  <div>
    <dt>{{ circuit.name }}</dt>
    <dd>{{ getPipeStat(circuit.key) }}</dd>
  </div>
</template>

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

      <div class="flow-section-title flow-section-title--button">
  <span>Designação</span>

  <button
    type="button"
    class="section-collapse-button"
    @click="toggleValveDesignationPanel"
  >
    {{ isValveDesignationPanelOpen ? '−' : '+' }}
  </button>
</div>

<p class="connection-note valve-name-line">
  Válvula: {{ selectedValveDesignation }}
</p>

<div v-if="isValveDesignationPanelOpen">
  <label class="flow-cycle-config">
    <span>Selecionar válvula para editar</span>

    <select
      v-model="selectedValveDesignationKey"
      @change="selectValveDesignationFromDropdown"
    >
      <option value="">
        Selecionar válvula...
      </option>

      <option
        v-for="option in getValveDesignationOptions()"
        :key="option.key"
        :value="option.key"
      >
        {{ option.label }}
      </option>
    </select>
  </label>

  <p class="connection-note">
    Nome original: {{ selectedValveOriginalDesignation || 'nenhuma válvula selecionada' }}
  </p>

  <div class="flow-section-title flow-section-title--button valve-rename-title">
  <span>Renomear válvula</span>

  <button
    type="button"
    class="section-collapse-button"
    @click="toggleValveRenamePanel"
  >
    {{ isValveRenamePanelOpen ? '−' : '+' }}
  </button>
</div>

<div v-if="isValveRenamePanelOpen">
  <label class="flow-cycle-config">
    <span>Novo nome da válvula</span>

    <input
      v-model="pendingValveDesignation"
      type="text"
      placeholder="Ex: Válvula 1"
    />
  </label>

  <div class="flow-actions flow-actions--secondary">
    <button type="button" @click="saveSelectedValveDesignation">
      Guardar nome
    </button>

    <button
      type="button"
      class="flow-button--danger"
      @click="resetSelectedValveDesignationToOriginal"
    >
      Repor nome original
    </button>
  </div>

  <p class="connection-note">
    Se deixares o campo vazio e guardares, a válvula volta ao nome original.
  </p>
</div>
</div>

<p class="connection-note">
  Estado: {{ getSelectedValveStateLabel() }} · {{ getSelectedValveAssociationStatusLabel() }}
</p>

<div class="flow-section-title flow-section-title--button valve-details-title">
  <span>Detalhes da associação</span>
  <button
    type="button"
    class="section-collapse-button"
    @click="toggleValveAssociationDetails"
  >
    {{ isValveAssociationDetailsOpen ? '−' : '+' }}
  </button>
</div>

<div v-if="isValveAssociationDetailsOpen" class="valve-association-summary">
  <p>
    <strong>Caminho:</strong>
    {{ getSelectedValveControlledRouteLabel() }}
  </p>

  <p>
    <strong>Tubo inicial:</strong>
    {{ getSelectedValveStartPipeLabel() }}
  </p>

  <p>
    <strong>Tubos:</strong>
    {{ getSelectedValveControlledPipeCountLabel() }}
  </p>

  <p>
    <strong>Troca:</strong>
    {{ getSelectedValveSwitchModeStatusLabel() }}
  </p>
</div>


<div class="flow-section-title">
  Ações da válvula
</div>

<div class="valve-actions-layout">
  <div class="valve-normal-state-badge">
    {{ getSelectedValveNormalTypeLabel() }}
  </div>

  <div class="flow-actions flow-actions--single valve-actions-buttons">
  <button
    type="button"
    :class="[
      'valve-switch-button',
      isSelectedValveInInverseState() ? 'valve-switch-button--active' : ''
    ]"
    @click="toggleSelectedValvesNormalInverseState"
  >
    {{ getSelectedValveSwitchLabel() }}
  </button>
</div>
</div>

<div class="flow-section-title">
  Associação da válvula ao caminho
</div>

<label class="flow-cycle-config">
  <span>Caminho controlado pela válvula</span>
  <select
    v-model="selectedValveAssociationRouteId"
    @change="highlightSelectedValveAssociationRoute"
  >
    <option value="">
      Selecionar caminho...
    </option>

    <option
      v-for="route in savedRoutes"
      :key="`valve-route-${route.id}`"
      :value="route.id"
    >
      {{ route.name }} - {{ getRouteCircuitDisplayLabel(route) }}
    </option>
  </select>
</label>

<label class="flow-cycle-config">
  <span>Troca de circuito a jusante</span>
  <select v-model="selectedValveSwitchMode">
    <option value="none">
      Não trocar
    </option>
    <option value="switchToSupply">
      Trocar para ida
    </option>
    <option value="switchToReturn">
      Trocar para retorno
    </option>
  </select>
</label>

<div class="flow-actions flow-actions--secondary">
  <button type="button" @click="linkSelectedPipesToPreparedValve">
    Associar tubo a jusante
  </button>

  <button
    type="button"
    class="flow-button--danger"
    @click="removeSelectedValvePipeLink"
  >
    Remover associação
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
         <template v-if="hasLoadedModel">
  <div class="flow-section-title flow-section-title--button">
    <span>Resumo da central</span>

    <button
      type="button"
      class="section-collapse-button"
      @click="toggleCentralSummary"
    >
      {{ isCentralSummaryOpen ? '−' : '+' }}
    </button>
  </div>

  <div v-if="isCentralSummaryOpen" class="central-summary"> 
  <div class="central-summary__item">
    <span>Ciclos</span>
    <strong>{{ waterCycleCount }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Elementos definidos</span>
    <strong>{{ countDefinedMepElements() }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Tubos classificados</span>
    <strong>{{ countMepElementsByType('pipe') }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Válvulas</span>
    <strong>{{ countValveElements() }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Boosters</span>
    <strong>{{ countMepElementsByType('booster') }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Reservatórios</span>
    <strong>{{ countReservoirElements() }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Caminhos guardados</span>
    <strong>{{ savedRoutes.length }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Caminhos visíveis</span>
    <strong>{{ countVisibleSavedRoutes() }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Caminhos ocultos</span>
    <strong>{{ countHiddenSavedRoutes() }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Caminhos protegidos</span>
    <strong>{{ countLockedSavedRoutes() }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Tubos marcados em circuitos</span>
    <strong>{{ pipeStats.total }}</strong>
  </div>

  <div class="central-summary__item">
      <span>Bloqueios ativos</span>
      <strong>{{ blockedCount }}</strong>
    </div>
  </div>
</template>
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

type PipeCircuit = string;
type CycleCircuitKind =
  | "hotSupply"
  | "coldSupply"
  | "hotReturn"
  | "coldReturn"
  | "extra";

type CycleCircuitDefinition = {
  key: PipeCircuit;
  cycleNumber: number;
  kind: CycleCircuitKind;
  name: string;
  color: string;
  defaultColor?: string;
  locked?: boolean;
  lockedDefault?: boolean;
};
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
  locked?: boolean;
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

type ValveSwitchMode =
  | "none"
  | "switchToSupply"
  | "switchToReturn";

type ValveControlledPipeLink = FlowNode & {
  temperature: PipeCircuit;
  routeId: string;
  switchMode?: ValveSwitchMode;
  targetTemperature?: PipeCircuit;
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
const discardRouteMessage = ref("");
const waterCycleCount = ref(3);
const pendingWaterCycleCount = ref(3);
const cycleNames = reactive<Record<string, string>>({});
const activeCycleNumber = ref(1);
const cycleCircuitDefinitions = reactive<CycleCircuitDefinition[]>([]);
const isCycleCircuitPanelOpen = ref(false);
const pendingCycleCircuitKind = ref<CycleCircuitKind>("extra");
const pendingCycleCircuitName = ref("");
const pendingCycleCircuitColor = ref("#2e7d32");
const selectedCycleCircuitKey = ref("");
const isIfcPanelCollapsed = ref(false);
const isCycleNamesPanelOpen = ref(false);
const isSavedRoutesPanelOpen = ref(true);
const highlightedSavedRouteId = ref<string | null>(null);
const isCentralSummaryOpen = ref(false);
const selectedCount = ref(0);
const selectedMepElementInfo = ref("Nenhum elemento classificado selecionado.");
const selectedValveDesignation = ref("nenhuma válvula selecionada");
const pendingValveDesignation = ref("");
const selectedValveOriginalDesignation = ref("");
const selectedValveDesignationKey = ref("");
const highlightedValveFromDropdown = ref<FlowNode | null>(null);
const isValveDesignationPanelOpen = ref(false);
const isValveRenamePanelOpen = ref(false);
const isValveAssociationDetailsOpen = ref(false);
const valveOriginalDesignations = reactive<Record<string, string>>({});
const hasLoadedModel = ref(false);
const isElementPanelMinimized = ref(true);
const isFlowControlsPanelMinimized = ref(true);
const isSimulationControlPanelMinimized = ref(true);
const routeStartLabel = ref("nenhum");
const routeEndLabel = ref("nenhum");
const blockedCount = ref(0);
const pipeStats = reactive<Record<string, number>>({
  supply: 0,
  return: 0,
  total: 0,
});

const MEP_ELEMENTS_STORAGE_KEY = "bastto-viewer-mep-elements";
const ROUTES_STORAGE_KEY = "bastto-viewer-routes";
const WATER_CYCLE_COUNT_STORAGE_KEY =
  "bastto-viewer-water-cycle-count";
const WATER_CYCLE_NAMES_STORAGE_KEY =
  "bastto-viewer-water-cycle-names";
const CYCLE_CIRCUITS_STORAGE_KEY =
  "bastto-viewer-cycle-circuits";
const REVERSED_DIRECTIONS_STORAGE_KEY =
  "bastto-viewer-reversed-directions";
const HIDDEN_FLOW_ARROWS_STORAGE_KEY =
  "bastto-viewer-hidden-flow-arrows";
const SYNCED_PIPE_DIRECTIONS_STORAGE_KEY =
  "bastto-viewer-synced-pipe-directions";
const VALVE_PIPE_LINKS_STORAGE_KEY =
  "bastto-viewer-valve-pipe-links";

let world: any;
let serializer: FRAGS.IfcImporter;
let fragmentManager: OBC.FragmentsManager;
let bimGridPanel: HTMLElement | null = null;
let bimGridViewport: HTMLElement | null = null;
let fragmentBytes: ArrayBuffer | null = null;
let animationFrame = 0;

const loadedModels = new Map<string, FRAGS.FragmentsModel>();
const flowGroup = new THREE.Group();
const pipeParticles: PipeParticle[] = [];
const staticFlowObjects: StaticFlowObject[] = [];
const selectedItems: SelectionMap = new Map();
const flowConnections = reactive<FlowConnection[]>([]);
const currentRouteConnections = reactive<FlowConnection[]>([]);
const routeWaypoints = reactive<FlowNode[]>([]);
const manualRouteNodes = reactive<FlowNode[]>([]);
const isManualRouteRecording = ref(false);
const mepElements = reactive<Record<string, MepElement>>({});
const savedRoutes = reactive<SavedRoute[]>([]);
let routeStart: FlowNode | null = null;
let routeEnd: FlowNode | null = null;
let ignoredManualRouteNodeAfterRemove: FlowNode | null = null;
const blockedPipes: SelectionMap = new Map();
const valveBlockedPipeLinks = new Map<string, ValveControlledPipeLink[]>();
const valveControlledPipeLinks = new Map<string, ValveControlledPipeLink[]>();
const blockedRoutePipes = new Set<string>();
const selectedValveForPipeLink = ref<FlowNode | null>(null);
const selectedValveAssociationRouteId = ref("");
const selectedValveSwitchMode = ref<ValveSwitchMode>("none");
const manualAssignments = reactive<Record<string, SelectionMap>>({});
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

const circuitMaterialCache = new Map<string, THREE.MeshBasicMaterial>();

function getCircuitColor(circuit: PipeCircuit) {
  const definition = getCycleCircuitDefinition(circuit);

  if (definition) {
    return Number(definition.color.replace("#", "0x"));
  }

  const cycleNumber = getCircuitCycleNumber(circuit);

  if (isSupplyCircuit(circuit)) {
    return Number(getDefaultSupplyColor(cycleNumber).replace("#", "0x"));
  }

  if (isReturnCircuit(circuit)) {
    return Number(getDefaultReturnColor(cycleNumber).replace("#", "0x"));
  }

  return Number(getDefaultExtraColor(cycleNumber).replace("#", "0x"));
}

function getCircuitMaterial(circuit: PipeCircuit) {
  const cachedMaterial = circuitMaterialCache.get(circuit);

  if (cachedMaterial) {
    return cachedMaterial;
  }

  const material = new THREE.MeshBasicMaterial({
    color: getCircuitColor(circuit),
    transparent: true,
    opacity: 0.9,
    depthTest: false,
  });

  circuitMaterialCache.set(circuit, material);

  return material;
}

onMounted(async () => {
  if (!containerRef.value) return;

  discardRouteMessage.value = "";

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
});

  fragmentManager.list.onItemSet.add(async ({ value: model }) => {
  model.useCamera(world?.camera.three as any);
  world?.scene.three.add(model.object);
  loadedModels.set(model.modelId, model);

  hasLoadedModel.value = true;

  await fragmentManager.core.update(true);

  if (savedRoutes.length) {
  await applyAllSavedRoutes();
  await clearBlockedPipes();
} else {
  flowMessage.value =
    `Modelo carregado: ${model.modelId}. Seleciona tubos e atribui os circuitos.`;
}
});

loadWaterCycleCountFromStorage();
loadCycleNamesFromStorage();
loadCycleCircuitDefinitionsFromStorage();
loadMepElementsFromStorage();
loadRoutesFromStorage();
loadReversedDirectionsFromStorage();
loadSyncedPipeDirectionsFromStorage();
loadHiddenFlowArrowsFromStorage();
loadValvePipeLinksFromStorage();

localStorage.removeItem("bastto-viewer-route-groups");

  createBimPanel(components, viewport);
  animateFlow();
});

onBeforeUnmount(() => {
  cancelAnimationFrame(animationFrame);
  clearFlowLayer();
});

function applyIfcPanelLayout() {
  const app = document.getElementById("appGrid") as BUI.Grid<["main"]> | null;

  if (!app || !bimGridViewport || !bimGridPanel) {
    return;
  }

  bimGridPanel.style.opacity = isIfcPanelCollapsed.value ? "0" : "1";
  bimGridPanel.style.pointerEvents = isIfcPanelCollapsed.value ? "none" : "auto";
  bimGridPanel.style.overflow = "hidden";

  app.layouts = {
    main: {
      template: isIfcPanelCollapsed.value
        ? `
          "panel viewport"
          / 0rem 1fr
        `
        : `
          "panel viewport"
          / 23rem 1fr
        `,
      elements: {
        panel: bimGridPanel,
        viewport: bimGridViewport,
      },
    },
  };

  app.layout = "main";

  if (fragmentManager) {
    setTimeout(() => {
      void fragmentManager.core.update(true);
    }, 0);
  }
}

function toggleIfcPanelCollapsed() {
  isIfcPanelCollapsed.value = !isIfcPanelCollapsed.value;
  applyIfcPanelLayout();
}

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

if (isManualRouteRecording.value && manualRouteNodes.length > 0) {
  await updateManualRoutePreviewHighlight();
  return;
}

await syncSelectedValveAssociationRoute();
});

  highlighter.events.select.onClear.add(async () => {
  selectedItems.clear();
  selectedCount.value = 0;
  selectedMepElementInfo.value = "Nenhum elemento classificado selecionado.";

  updatePropertiesTable({
    modelIdMap: {},
  });

  if (isManualRouteRecording.value) {
    return;
  }

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

  bimGridPanel = panel as HTMLElement;
bimGridViewport = viewport;

applyIfcPanelLayout();
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
  "Modelo carregado. Define manualmente os circuitos de ida e retorno.";
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
    for (const existingCircuit of getAllKnownCircuitKeys()) {
  getAssignmentSet(existingCircuit, modelId).delete(id);
}

    targetSet.add(id);
  }
}

  updateManualStats();
  await rebuildManualFlowLayer();
  flowMessage.value =
  selectedCount.value +
  " elemento(s) marcados como " +
  getCircuitLabel(circuit) +
  ".";
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

  for (const circuit of getAllKnownCircuitKeys()) {
  await addAssignmentsToScene(circuit);
}

  updateManualStats();

  const shouldKeepAnimating =
  !isFlowManuallyPaused.value &&
  (isManualFlowAnimationRunning.value || isCentralSimulationRunning.value);

isFlowing.value = shouldKeepAnimating && pipeParticles.length > 0;

  await fragmentManager.core.update(true);

}

async function addAssignmentsToScene(temperature: PipeCircuit) {
  const assignmentMap = manualAssignments[temperature];

  if (!assignmentMap) {
    return;
  }

  for (const [modelId, idsSet] of assignmentMap) {
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

    await model.highlight(
  ids,
  createHighlight(
    getCircuitColor(temperature),
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

if (shouldHidePipeForCircuit(modelId, localId, temperature)) {
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

      for (const circuit of getAllKnownCircuitKeys()) {
  getAssignmentSet(circuit, modelId).delete(localId);
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
  
  updateManualStats();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    clearFlowVisuals();
    await fragmentManager.core.update(true);
  }

  flowMessage.value = clearedCount
  ? clearedCount + " marca(s) selecionada(s) removida(s)."
  : "Nenhuma marca selecionada para remover.";
}

async function clearManualAssignments() {
  clearFlowVisuals();

  for (const [modelId, model] of loadedModels) {
    const ids = getAllKnownCircuitKeys().flatMap((circuit) => [
  ...getAssignmentSet(circuit, modelId),
]);
    if (ids.length) await model.resetHighlight(ids);
  }

  for (const circuit of getAllKnownCircuitKeys()) {
  manualAssignments[circuit]?.clear();
}

  flowConnections.splice(0);
  currentRouteConnections.splice(0);
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
  for (const circuit of getAllKnownCircuitKeys()) {
  manualAssignments[circuit]?.clear();
}
  flowConnections.splice(0);
  currentRouteConnections.splice(0);
  routeWaypoints.splice(0);
  routeStart = null;
  routeEnd = null;
  routeStartLabel.value = "nenhum";
  routeEndLabel.value = "nenhum";
}

function startManualRouteRecording() {
  discardRouteMessage.value = "";
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

  if (
    ignoredManualRouteNodeAfterRemove &&
    isSameNode(ignoredManualRouteNodeAfterRemove, node)
  ) {
    ignoredManualRouteNodeAfterRemove = null;
    return;
  }

  const lastNode = manualRouteNodes[manualRouteNodes.length - 1];

  if (lastNode && isSameNode(lastNode, node)) {
    return;
  }

  manualRouteNodes.push(node);
  void updateManualRoutePreviewHighlight();

  flowMessage.value =
    "Ponto manual adicionado. Total: " +
    manualRouteNodes.length +
    ".";
}

async function removeLastManualRouteNode() {
  if (manualRouteNodes.length === 0) {
    flowMessage.value = "Não há tubos para remover do caminho manual.";
    return;
  }

  const removedNode = manualRouteNodes[manualRouteNodes.length - 1];
  ignoredManualRouteNodeAfterRemove = removedNode;
  manualRouteNodes.splice(manualRouteNodes.length - 1, 1);

  const model = loadedModels.get(removedNode.modelId);

  if (model) {
    await model.resetHighlight([removedNode.localId]);
  }

  if (manualRouteNodes.length > 0) {
    await updateManualRoutePreviewHighlight();
  } else {
    await fragmentManager.core.update(true);
  }

  flowMessage.value =
    "Último tubo removido do caminho manual. Total: " +
    manualRouteNodes.length +
    ".";
}

async function updateManualRoutePreviewHighlight() {
  const idsByModel = new Map<string, number[]>();

  for (const node of manualRouteNodes) {
    if (!idsByModel.has(node.modelId)) {
      idsByModel.set(node.modelId, []);
    }

    idsByModel.get(node.modelId)?.push(node.localId);
  }

  for (const [modelId, ids] of idsByModel) {
    const model = loadedModels.get(modelId);

    if (!model || !ids.length) {
      continue;
    }

    await model.highlight(
      ids,
      createHighlight(0x8fd3ff, "manual-route-preview"),
    );
  }

  await fragmentManager.core.update(true);
}

function setRouteStart() {
  discardRouteMessage.value = "";
  const node = getFirstSelectedNode();
  if (!node) {
    flowMessage.value = "Seleciona primeiro o tubo/ponto inicial.";
    return;
  }

  routeStart = node;
  routeStartLabel.value = formatNodeLabel(node);
  flowMessage.value = "Início definido. Agora seleciona o tubo final e clica em 2. Definir fim.";
}

function setRouteEnd() {
  discardRouteMessage.value = "";

  const node = getFirstSelectedNode();
  if (!node) {
    flowMessage.value = "Seleciona primeiro o tubo/ponto final.";
    return;
  }

  routeEnd = node;
  routeEndLabel.value = formatNodeLabel(node);
  flowMessage.value =
  "Fim definido. Agora clica em 3. Criar caminho automático.";
}

async function createManualRouteFromSelection(temperature: PipeCircuit) {
  discardRouteMessage.value = "";

  if (manualRouteNodes.length < 2) {
    flowMessage.value =
      "Seleciona pelo menos dois tubos pela ordem do caminho manual.";
    return;
  }

  const path = [...manualRouteNodes];

  currentRouteConnections.splice(0);

  for (let index = 0; index < path.length - 1; index++) {
    const connection: FlowConnection = {
      from: path[index],
      to: path[index + 1],
      temperature,
    };

    currentRouteConnections.push(connection);
    addFlowConnectionIfMissing(connection);
  }

  assignPathToTemperature(path, temperature);
  updateManualStats();

  await rebuildManualFlowLayer();

  manualRouteNodes.splice(0);
  isManualRouteRecording.value = false;

  flowMessage.value =
    "Caminho manual " +
    getCircuitLabel(temperature) +
    " criado com " +
    path.length +
    " tubo(s).";
}

async function createAutoRoute(temperature: PipeCircuit) {
  discardRouteMessage.value = "";
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

    currentRouteConnections.splice(0);

for (let index = 0; index < path.length - 1; index++) {
  const connection: FlowConnection = {
    from: path[index],
    to: path[index + 1],
    temperature,
  };

  currentRouteConnections.push(connection);
  addFlowConnectionIfMissing(connection);
}

    assignPathToTemperature(path, temperature);
    updateManualStats();
    await rebuildManualFlowLayer();
    flowMessage.value =
  "Caminho manual " +
  getCircuitLabel(temperature) +
  " criado com " +
  path.length +
  " tubo(s).";
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

    for (const existingCircuit of getAllKnownCircuitKeys()) {
  getAssignmentSet(existingCircuit, node.modelId).delete(node.localId);
}

    getAssignmentSet(temperature, node.modelId).add(node.localId);
  }
}

function saveCurrentRoute() {
  if (!currentRouteConnections.length) {
    flowMessage.value = "Cria primeiro um caminho antes de o guardar.";
    return;
  }

  const temperature = currentRouteConnections[0].temperature;

  const path: FlowNode[] = [
    currentRouteConnections[0].from,
    ...currentRouteConnections.map((connection) => connection.to),
  ];

  const circuitLabel = getCircuitLabel(temperature);
  const routeNumber = getNextRouteNumberForCircuit(temperature);

  savedRoutes.push({
    id: crypto.randomUUID(),
    name:
      capitalizeFirstLetter(circuitLabel) +
      " - Caminho " +
      routeNumber,
    temperature,
    path,
  });

  saveRoutesToStorage();

  discardRouteMessage.value = "";
  flowMessage.value = "Caminho guardado com sucesso.";
}

async function discardCurrentRoute() {
  if (!currentRouteConnections.length && !manualRouteNodes.length) {
  discardRouteMessage.value = "Não existe caminho atual para descartar.";
  flowMessage.value = "Não existe caminho atual para descartar.";
  return;
}

  const nodesToClear = new Map<string, FlowNode>();

  for (const connection of currentRouteConnections) {
    nodesToClear.set(nodeKey(connection.from), connection.from);
    nodesToClear.set(nodeKey(connection.to), connection.to);
  }

  for (const node of manualRouteNodes) {
    nodesToClear.set(nodeKey(node), node);
  }

  for (let index = flowConnections.length - 1; index >= 0; index--) {
    const shouldRemove = currentRouteConnections.some((connection) =>
      isSameConnection(flowConnections[index], connection),
    );

    if (shouldRemove) {
      flowConnections.splice(index, 1);
    }
  }

  for (const node of nodesToClear.values()) {
    for (const circuit of getAllKnownCircuitKeys()) {
      getAssignmentSet(circuit, node.modelId).delete(node.localId);
    }
  }

  currentRouteConnections.splice(0);
  manualRouteNodes.splice(0);
  routeWaypoints.splice(0);

  routeStart = null;
  routeEnd = null;
  routeStartLabel.value = "nenhum";
  routeEndLabel.value = "nenhum";
  isManualRouteRecording.value = false;

  updateManualStats();

  if (savedRoutes.length) {
    await applyAllSavedRoutes();
  } else if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    clearFlowVisuals();
    await fragmentManager.core.update(true);
  }

  discardRouteMessage.value =
  "Caminho atual descartado. Caminhos guardados mantidos.";

flowMessage.value =
  "Caminho atual descartado. Caminhos guardados mantidos.";
}

async function deleteSavedRoute(routeId: string) {
  const index = savedRoutes.findIndex(
    (route) => route.id === routeId,
  );

  if (index === -1) return;

  const route = savedRoutes[index];

if (route.locked) {
  flowMessage.value =
    "O caminho \"" +
    route.name +
    "\" está protegido. Desprotege primeiro para apagar.";
  return;
}

if (highlightedSavedRouteId.value === routeId) {
  highlightedSavedRouteId.value = null;
}

savedRoutes.splice(index, 1);

  const idsByModel = new Map<string, number[]>();

  for (const node of route.path) {
    for (const circuit of getAllKnownCircuitKeys()) {
  getAssignmentSet(circuit, node.modelId).delete(node.localId);
}

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

async function restoreFlowVisualsAfterRouteHighlight() {
  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
    return;
  }

  await fragmentManager.core.update(true);
}

async function toggleSavedRouteHighlight(route: SavedRoute) {
  const loadedModelIds = [...loadedModels.keys()];

  if (!loadedModelIds.length) {
    flowMessage.value = "Carrega primeiro o IFC antes de realçar um caminho.";
    return;
  }

  if (highlightedSavedRouteId.value === route.id) {
    highlightedSavedRouteId.value = null;

    await resetRoutePathHighlight(route);
    await restoreFlowVisualsAfterRouteHighlight();

    flowMessage.value =
  "Realce do caminho \"" +
  route.name +
  "\" removido.";
    return;
  }

  const previousRoute = savedRoutes.find(
    (savedRoute) => savedRoute.id === highlightedSavedRouteId.value,
  );

  if (previousRoute) {
    await resetRoutePathHighlight(previousRoute);
    await restoreFlowVisualsAfterRouteHighlight();
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

  const idsByModel = new Map<string, number[]>();

  for (const node of adaptedPath) {
    if (!idsByModel.has(node.modelId)) {
      idsByModel.set(node.modelId, []);
    }

    idsByModel.get(node.modelId)?.push(node.localId);
  }

  for (const [modelId, ids] of idsByModel) {
    const model = loadedModels.get(modelId);

    if (!model || !ids.length) {
      continue;
    }

    await model.highlight(
      ids,
      createHighlight(0x00e5ff, "saved-route-highlight-" + route.id),
    );
  }

  highlightedSavedRouteId.value = route.id;

  await fragmentManager.core.update(true);

  flowMessage.value =
  "Caminho \"" +
  route.name +
  "\" realçado.";
}

async function setSavedRouteVisibility(routeId: string, shouldShow: boolean) {
  const route = savedRoutes.find((savedRoute) => savedRoute.id === routeId);

  if (!route) {
    return;
  }

  route.hidden = !shouldShow;

  saveRoutesToStorage();

  await resetRoutePathHighlight(route);

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    await fragmentManager.core.update(true);
  }

  if (shouldShow) {
  flowMessage.value = "Caminho \"" + route.name + "\" visível.";
  return;
}

flowMessage.value = "Caminho \"" + route.name + "\" oculto.";
}

async function applySavedRoute(route: SavedRoute) {
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

async function reverseSavedRoute(routeId: string) {
  const route = savedRoutes.find(
    (savedRoute) => savedRoute.id === routeId,
  );

  if (!route) return;

  if (route.locked) {
  flowMessage.value =
    `O caminho "${route.name}" está protegido. Desprotege primeiro para inverter.`;
  return;
}

  const reversedPath = [...route.path].reverse();

  route.path.splice(
    0,
    route.path.length,
    ...reversedPath,
  );

  toggleSyncedPipesForPath(route.path);

  saveRoutesToStorage();

  if (loadedModels.size) {
    await applySavedRoute(route);
  }

  flowMessage.value =
    `Sentido do caminho "${route.name}" invertido.`;
}

function toggleSavedRouteProtection(routeId: string) {
  const route = savedRoutes.find(
    (savedRoute) => savedRoute.id === routeId,
  );

  if (!route) {
    return;
  }

  route.locked = !route.locked;

  saveRoutesToStorage();

  if (route.locked) {
    flowMessage.value = `Caminho "${route.name}" protegido.`;
    return;
  }

  flowMessage.value = `Caminho "${route.name}" desprotegido.`;
}

function renameSavedRoute(routeId: string) {
  const route = savedRoutes.find(
    (savedRoute) => savedRoute.id === routeId,
  );

  if (!route) return;

  if (route.locked) {
  flowMessage.value =
    `O caminho "${route.name}" está protegido. Desprotege primeiro para renomear.`;
  return;
}

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

function saveValvePipeLinksToStorage() {
  const data = [...valveControlledPipeLinks.entries()].map(
    ([valveKey, linkedPipes]) => ({
      valveKey,
      linkedPipes,
    }),
  );

  localStorage.setItem(
    VALVE_PIPE_LINKS_STORAGE_KEY,
    JSON.stringify(data),
  );
}

function loadValvePipeLinksFromStorage() {
  const saved = localStorage.getItem(VALVE_PIPE_LINKS_STORAGE_KEY);

  if (!saved) {
    return;
  }

  try {
    const parsed = JSON.parse(saved) as {
      valveKey: string;
      linkedPipes: ValveControlledPipeLink[];
    }[];

    valveControlledPipeLinks.clear();

    for (const item of parsed) {
      valveControlledPipeLinks.set(item.valveKey, item.linkedPipes);
    }
  } catch (error) {
    console.error("Erro ao carregar associações de válvulas:", error);
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

async function defineSelectedElementsAs(elementType: MepElementType) {
  if (!selectedCount.value) {
    flowMessage.value = "Seleciona primeiro um ou mais elementos no modelo.";
    return;
  }

  for (const [modelId, ids] of selectedItems) {
    for (const localId of ids) {
      const key = elementKey(modelId, localId);

      const defaultState = isValveElementType(elementType)
        ? getNormalValveState(elementType)
        : mepElements[key]?.state;

      mepElements[key] = {
        modelId,
        localId,
        elementType,
        circuitType: mepElements[key]?.circuitType ?? "unknown",
        state: defaultState,
      };
    }
  }

  saveMepElementsToStorage();

  if (elementType === "normallyClosedValve") {
    await setSelectedValvesState("closed");

    flowMessage.value =
      `${selectedCount.value} elemento(s) definidos como ${getElementTypeLabel(elementType)}. ` +
      `Válvula(s) NF reposta(s) ao estado normal fechado.`;

    return;
  }

  if (elementType === "normallyOpenValve") {
    await setSelectedValvesState("open");

    flowMessage.value =
      `${selectedCount.value} elemento(s) definidos como ${getElementTypeLabel(elementType)}. ` +
      `Válvula(s) NA reposta(s) ao estado normal aberto.`;

    return;
  }

  flowMessage.value =
    `${selectedCount.value} elemento(s) definidos como ${getElementTypeLabel(elementType)}.`;
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

      for (const circuit of getAllKnownCircuitKeys()) {
  getAssignmentSet(circuit, modelId).delete(localId);
}
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

async function deleteAllElementDefinitions() {
  const elements = Object.values(mepElements);

  if (!elements.length) {
    flowMessage.value = "Não existem definições de elementos para apagar.";
    return;
  }

  const shouldContinue = confirm(
    "Esta ação vai apagar todas as definições dos elementos. Deseja continuar?",
  );

  if (!shouldContinue) {
    flowMessage.value = "Remoção de todas as definições cancelada.";
    return;
  }

  const idsByModel = new Map<string, number[]>();

  for (const element of elements) {
    const { modelId, localId } = element;

    for (const circuit of getAllKnownCircuitKeys()) {
      getAssignmentSet(circuit, modelId).delete(localId);
    }

    reversedPipeDirections.get(modelId)?.delete(localId);

    if (reversedPipeDirections.get(modelId)?.size === 0) {
      reversedPipeDirections.delete(modelId);
    }

    blockedPipes.get(modelId)?.delete(localId);

    if (blockedPipes.get(modelId)?.size === 0) {
      blockedPipes.delete(modelId);
    }

    if (!idsByModel.has(modelId)) {
      idsByModel.set(modelId, []);
    }

    idsByModel.get(modelId)?.push(localId);
  }

  for (const key of Object.keys(mepElements)) {
    delete mepElements[key];
  }

  valveBlockedPipeLinks.clear();
  valveControlledPipeLinks.clear();
blockedRoutePipes.clear();
localStorage.removeItem(VALVE_PIPE_LINKS_STORAGE_KEY);

  for (const [modelId, ids] of idsByModel) {
    const model = loadedModels.get(modelId);

    if (model && ids.length) {
      await model.resetHighlight(ids);
    }
  }

  updateBlockedCount();
  updateManualStats();

  saveMepElementsToStorage();
  saveReversedDirectionsToStorage();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    clearFlowVisuals();
    await fragmentManager.core.update(true);
  }

  flowMessage.value =
    `${elements.length} definição(ões) de elemento apagada(s).`;
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

function getValveDesignationOptions() {
  return Object.values(mepElements)
    .filter((element) => isValveElementType(element.elementType))
    .map((element) => {
      const key = elementKey(element.modelId, element.localId);
      const valveKey = nodeKey({
        modelId: element.modelId,
        localId: element.localId,
      });

      const originalName =
        valveOriginalDesignations[key] || `Válvula #${element.localId}`;

      const displayName = element.name || originalName;

      const hasAssociation =
        valveControlledPipeLinks.has(valveKey) ||
        [...valveControlledPipeLinks.keys()].some((storedValveKey) => {
          const [, storedLocalId] = storedValveKey.split(":");
          return Number(storedLocalId) === element.localId;
        });

      return {
        key,
        label: hasAssociation
          ? `${displayName} - associada`
          : `${displayName} - sem associação`,
      };
    });
}

function getValveNodeFromDesignationKey(key: string): FlowNode | null {
  const separatorIndex = key.lastIndexOf(":");

  if (separatorIndex === -1) {
    return null;
  }

  const modelId = key.slice(0, separatorIndex);
  const localId = Number(key.slice(separatorIndex + 1));

  if (!modelId || !Number.isFinite(localId)) {
    return null;
  }

  return {
    modelId,
    localId,
  };
}

async function getOriginalDesignationForNode(node: FlowNode) {
  const key = elementKey(node.modelId, node.localId);

  if (valveOriginalDesignations[key]) {
    return valveOriginalDesignations[key];
  }

  const model = loadedModels.get(node.modelId);
  let originalName = "";

  if (model) {
    const [itemData] = await model.getItemsData([node.localId], {
      attributesDefault: true,
      relationsDefault: {
        attributes: true,
        relations: false,
      },
    });

    originalName = getItemNameFromData(itemData);
  }

  const fallbackName = originalName || `Válvula #${node.localId}`;

  valveOriginalDesignations[key] = fallbackName;

  return fallbackName;
}

async function syncValveDesignationPanelFromNode(node: FlowNode) {
  const key = elementKey(node.modelId, node.localId);
  const element = mepElements[key];

  if (!element || !isValveElementType(element.elementType)) {
    selectedValveDesignation.value = "nenhuma válvula selecionada";
    selectedValveOriginalDesignation.value = "";
    pendingValveDesignation.value = "";
    selectedValveDesignationKey.value = "";
    return;
  }

  const originalName = await getOriginalDesignationForNode(node);

  selectedValveDesignationKey.value = key;
  selectedValveOriginalDesignation.value = originalName;
  selectedValveDesignation.value = element.name || originalName;
  pendingValveDesignation.value = element.name || "";
}

async function selectValveDesignationFromDropdown() {
  const valveNode = getValveNodeFromDesignationKey(
    selectedValveDesignationKey.value,
  );

  if (!valveNode) {
  selectedValveDesignation.value = "nenhuma válvula selecionada";
  selectedValveOriginalDesignation.value = "";
  pendingValveDesignation.value = "";
  selectedValveForPipeLink.value = null;

  if (highlightedValveFromDropdown.value) {
    const previousModel = loadedModels.get(
      highlightedValveFromDropdown.value.modelId,
    );

    if (previousModel) {
      await previousModel.resetHighlight([
        highlightedValveFromDropdown.value.localId,
      ]);
    }

    highlightedValveFromDropdown.value = null;

    await fragmentManager.core.update(true);
  }

  return;
}

  await syncValveDesignationPanelFromNode(valveNode);

  selectedValveForPipeLink.value = valveNode;

  await highlightValveFromDropdown(valveNode);

  const linkedPipes = getLinkedPipesForValveNode(valveNode);
const associatedRouteId = linkedPipes[0]?.routeId;
selectedValveSwitchMode.value = linkedPipes[0]?.switchMode ?? "none";

  if (associatedRouteId) {
    selectedValveAssociationRouteId.value = associatedRouteId;

    const associatedRoute = savedRoutes.find(
      (route) => route.id === associatedRouteId,
    );

    if (
      associatedRoute &&
      highlightedSavedRouteId.value !== associatedRoute.id
    ) {
      await toggleSavedRouteHighlight(associatedRoute);
    }
  }

  flowMessage.value =
    "Válvula selecionada no dropdown e destacada no modelo.";
}

function getValveNodeForDesignationEditing() {
  return getValveNodeForControl();
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
  let originalElementName = "";

  if (model) {
    const [itemData] = await model.getItemsData([selectedNode.localId], {
      attributesDefault: true,
      relationsDefault: {
        attributes: true,
        relations: false,
      },
    });

    originalElementName = getItemNameFromData(itemData);
  }

  const typeLabel = getElementTypeLabel(definedElement.elementType);
  const fallbackOriginalName =
    originalElementName || `elemento #${selectedNode.localId}`;

  const displayName =
    definedElement.name || fallbackOriginalName;

  if (isValveElementType(definedElement.elementType)) {
    valveOriginalDesignations[key] =
      originalElementName || `Válvula #${selectedNode.localId}`;

    await syncValveDesignationPanelFromNode(selectedNode);
  }

  selectedMepElementInfo.value = `${typeLabel}: ${displayName}`;
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

function countLockedSavedRoutes() {
  return savedRoutes.filter((route) => route.locked).length;
}

function countHiddenSavedRoutes() {
  return savedRoutes.filter((route) => route.hidden).length;
}

function countVisibleSavedRoutes() {
  return savedRoutes.filter((route) => !route.hidden).length;
}

function countValveElements() {
  return (
    countMepElementsByType("isolationValve") +
    countMepElementsByType("normallyOpenValve") +
    countMepElementsByType("normallyClosedValve")
  );
}

function countReservoirElements() {
  return (
    countMepElementsByType("reservoirWithResistance") +
    countMepElementsByType("reservoirWithoutResistance")
  );
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

function isValveElementType(elementType: MepElementType) {
  return (
    elementType === "isolationValve" ||
    elementType === "normallyOpenValve" ||
    elementType === "normallyClosedValve"
  );
}

function getNormalValveState(elementType: MepElementType) {
  if (elementType === "normallyClosedValve") {
    return "closed";
  }

  return "open";
}

function getInverseValveState(elementType: MepElementType) {
  return getNormalValveState(elementType) === "closed" ? "open" : "closed";
}

function getSelectedValveElements() {
  const selectedValves: MepElement[] = [];

  for (const valveNode of getValveNodesForControl()) {
    const element = mepElements[elementKey(
      valveNode.modelId,
      valveNode.localId,
    )];

    if (!element || !isValveElementType(element.elementType)) {
      continue;
    }

    selectedValves.push(element);
  }

  return selectedValves;
}

function getSelectedValveInverseButtonLabel() {
  const selectedValves = getSelectedValveElements();

  if (!selectedValves.length) {
    return "Inverter estado normal";
  }

  const firstValve = selectedValves[0];
  const inverseState = getInverseValveState(firstValve.elementType);

  if (firstValve.elementType === "normallyClosedValve") {
    return inverseState === "open"
      ? "Abrir válvula NF"
      : "Fechar válvula NF";
  }

  if (firstValve.elementType === "normallyOpenValve") {
    return inverseState === "closed"
      ? "Fechar válvula NA"
      : "Abrir válvula NA";
  }

  return inverseState === "open" ? "Abrir válvula" : "Fechar válvula";
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
  const directlyBlockedPipes = [...blockedPipes.values()].reduce(
    (total, ids) => total + ids.size,
    0,
  );

  blockedCount.value = directlyBlockedPipes + blockedRoutePipes.size;
}

async function highlightSelectedValveAssociationRoute() {
  const selectedRoute = savedRoutes.find(
    (route) => route.id === selectedValveAssociationRouteId.value,
  );

  if (!selectedRoute) {
    const highlightedRoute = savedRoutes.find(
      (route) => route.id === highlightedSavedRouteId.value,
    );

    if (highlightedRoute) {
      await toggleSavedRouteHighlight(highlightedRoute);
    }

    return;
  }

  if (highlightedSavedRouteId.value === selectedRoute.id) {
    return;
  }

  await toggleSavedRouteHighlight(selectedRoute);
}

function getHighlightedRouteForValveAssociation() {
  const selectedRoute = savedRoutes.find(
    (route) => route.id === selectedValveAssociationRouteId.value,
  );

  if (selectedRoute) {
    return selectedRoute;
  }

  if (!highlightedSavedRouteId.value) {
    return null;
  }

  return savedRoutes.find(
    (route) => route.id === highlightedSavedRouteId.value,
  ) ?? null;
}

function getSelectedValveAssociationStatusLabel() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    return "nenhuma válvula selecionada";
  }

  const valveKey = nodeKey(valveNode);

  if (valveControlledPipeLinks.has(valveKey)) {
    return "válvula associada";
  }

  const hasAssociationByLocalId = [...valveControlledPipeLinks.keys()].some(
    (storedValveKey) => {
      const [, storedLocalId] = storedValveKey.split(":");
      return Number(storedLocalId) === valveNode.localId;
    },
  );

  if (hasAssociationByLocalId) {
    return "válvula associada";
  }

  return "sem associação";
}

function getSelectedValveControlledRouteLabel() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    return "nenhum";
  }

  const linkedPipes = getLinkedPipesForValveNode(valveNode);
  const routeId = linkedPipes[0]?.routeId;

  if (!routeId) {
    return "nenhum";
  }

  const route = savedRoutes.find((savedRoute) => savedRoute.id === routeId);

  if (!route) {
    return "caminho não encontrado";
  }

  return `${route.name} - ${getRouteCircuitDisplayLabel(route)}`;
}

function getSelectedValveStartPipeLabel() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    return "nenhum";
  }

  const linkedPipes = getLinkedPipesForValveNode(valveNode);
  const firstPipe = linkedPipes[0];

  if (!firstPipe) {
    return "nenhum";
  }

  return `#${firstPipe.localId}`;
}

function getSelectedValveControlledPipeCountLabel() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    return "0";
  }

  const linkedPipes = getLinkedPipesForValveNode(valveNode);

  return String(linkedPipes.length);
}

function getLinkedPipesForValveNode(valveNode: FlowNode) {
  const valveKey = nodeKey(valveNode);

  const exactLinkedPipes = valveControlledPipeLinks.get(valveKey);

  if (exactLinkedPipes) {
    return exactLinkedPipes;
  }

  return (
    [...valveControlledPipeLinks.entries()]
      .find(([storedValveKey]) => {
        const [, storedLocalId] = storedValveKey.split(":");
        return Number(storedLocalId) === valveNode.localId;
      })?.[1] ?? []
  );
}

function getSelectedValveSwitchModeStatusLabel() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    return "nenhuma";
  }

  const linkedPipes = getLinkedPipesForValveNode(valveNode);
  const switchMode = linkedPipes[0]?.switchMode ?? "none";

  return getValveSwitchModeLabel(switchMode);
}

async function syncSelectedValveAssociationRoute() {
  const valveNode = getFirstSelectedValveNode();

  if (!valveNode) {
    return;
  }

  selectedValveForPipeLink.value = valveNode;

  const linkedPipes = getLinkedPipesForValveNode(valveNode);
  const associatedRouteId = linkedPipes[0]?.routeId;

  if (!associatedRouteId) {
    return;
  }

  selectedValveAssociationRouteId.value = associatedRouteId;

  const associatedRoute = savedRoutes.find(
    (route) => route.id === associatedRouteId,
  );

  if (!associatedRoute) {
    return;
  }

  if (highlightedSavedRouteId.value === associatedRoute.id) {
    highlightedSavedRouteId.value = null;
  }

  await toggleSavedRouteHighlight(associatedRoute);
}

function getSelectedValveStateLabel() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    return "nenhuma válvula selecionada";
  }

  const element = mepElements[elementKey(
    valveNode.modelId,
    valveNode.localId,
  )];

  if (!element || !isValveElementType(element.elementType)) {
    return "nenhuma válvula selecionada";
  }

  if (element.state === "closed") {
    return "fechada";
  }

  if (element.state === "open") {
    return "aberta";
  }

  return "estado desconhecido";
}

function getSelectedValveNormalTypeLabel() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    return "Nenhuma válvula selecionada";
  }

  const element = mepElements[elementKey(
    valveNode.modelId,
    valveNode.localId,
  )];

  if (!element || !isValveElementType(element.elementType)) {
    return "Nenhuma válvula selecionada";
  }

  if (element.elementType === "normallyClosedValve") {
    return "Normalmente fechada";
  }

  if (element.elementType === "normallyOpenValve") {
    return "Normalmente aberta";
  }

  return "Válvula de corte";
}

async function saveSelectedValveDesignation() {
  const valveNode = getValveNodeForDesignationEditing();

  if (!valveNode) {
    alert("Seleciona primeiro a válvula que queres renomear.");

    flowMessage.value =
      "Seleciona primeiro a válvula que queres renomear.";

    return;
  }

  const key = elementKey(valveNode.modelId, valveNode.localId);
  const element = mepElements[key];

  if (!element || !isValveElementType(element.elementType)) {
    alert("Seleciona primeiro uma válvula NA ou NF.");

    flowMessage.value =
      "Seleciona primeiro uma válvula NA ou NF.";

    return;
  }

  const trimmedName = pendingValveDesignation.value.trim();

  if (!trimmedName) {
    const { name, ...elementWithoutCustomName } = element;

    mepElements[key] = {
      ...elementWithoutCustomName,
    };

    saveMepElementsToStorage();

    await syncValveDesignationPanelFromNode(valveNode);

    flowMessage.value =
      "Nome personalizado removido. A válvula voltou ao nome original.";

    return;
  }

  mepElements[key] = {
    ...element,
    name: trimmedName,
  };

  saveMepElementsToStorage();

  await syncValveDesignationPanelFromNode(valveNode);

  flowMessage.value =
    `Nome da válvula guardado: ${trimmedName}.`;
}

async function resetSelectedValveDesignationToOriginal() {
  const valveNode = getValveNodeForDesignationEditing();

  if (!valveNode) {
    alert("Seleciona primeiro a válvula que queres repor.");

    flowMessage.value =
      "Seleciona primeiro a válvula que queres repor.";

    return;
  }

  const key = elementKey(valveNode.modelId, valveNode.localId);
  const element = mepElements[key];

  if (!element || !isValveElementType(element.elementType)) {
    alert("Seleciona primeiro uma válvula NA ou NF.");

    flowMessage.value =
      "Seleciona primeiro uma válvula NA ou NF.";

    return;
  }

  const { name, ...elementWithoutCustomName } = element;

  mepElements[key] = {
    ...elementWithoutCustomName,
  };

  saveMepElementsToStorage();

  await syncValveDesignationPanelFromNode(valveNode);

  flowMessage.value =
    "Nome original da válvula reposto.";
}

function isValveAtNormalState(element: MepElement) {
  return element.state === getNormalValveState(element.elementType);
}

function isSelectedValveInInverseState() {
  const selectedValves = getSelectedValveElements();

  if (!selectedValves.length) {
    return false;
  }

  return selectedValves.some(
    (valve) => !isValveAtNormalState(valve),
  );
}

function getSelectedValveSwitchLabel() {
  const selectedValves = getSelectedValveElements();

  if (!selectedValves.length) {
    return "Alternar válvula";
  }

  const firstValve = selectedValves[0];

  if (isSelectedValveInInverseState()) {
    return "Repor estado normal";
  }

  if (firstValve.elementType === "normallyClosedValve") {
    return "Abrir válvula NF";
  }

  if (firstValve.elementType === "normallyOpenValve") {
    return "Fechar válvula NA";
  }

  return "Inverter estado normal";
}

async function toggleSelectedValvesNormalInverseState() {
  const selectedValves = getSelectedValveElements();

  if (!selectedValves.length) {
    alert("Seleciona primeiro a válvula que queres alternar.");

    flowMessage.value =
      "Seleciona primeiro a válvula que queres alternar.";

    return;
  }

  const shouldResetToNormal = isSelectedValveInInverseState();

  if (shouldResetToNormal) {
    await resetSelectedValvesToNormal();
    return;
  }

  await applyInverseNormalStateToSelectedValves();
}

function getFirstSelectedValveNode() {
  for (const [modelId, ids] of selectedItems) {
    for (const localId of ids) {
      if (isIsolationValve(modelId, localId)) {
        return {
          modelId,
          localId,
        };
      }
    }
  }

  return null;
}

function getValveNodesForControl() {
  const dropdownValveNode = getValveNodeFromDesignationKey(
    selectedValveDesignationKey.value,
  );

  if (dropdownValveNode) {
    return [dropdownValveNode];
  }

  const selectedValveNodes: FlowNode[] = [];

  for (const [modelId, ids] of selectedItems) {
    for (const localId of ids) {
      if (isIsolationValve(modelId, localId)) {
        selectedValveNodes.push({
          modelId,
          localId,
        });
      }
    }
  }

  return selectedValveNodes;
}

function getValveNodeForControl() {
  return (
    getValveNodeFromDesignationKey(selectedValveDesignationKey.value) ??
    getFirstSelectedValveNode()
  );
}

async function highlightValveFromDropdown(node: FlowNode) {
  if (highlightedValveFromDropdown.value) {
    const previousModel = loadedModels.get(
      highlightedValveFromDropdown.value.modelId,
    );

    if (previousModel) {
      await previousModel.resetHighlight([
        highlightedValveFromDropdown.value.localId,
      ]);
    }
  }

  const model = loadedModels.get(node.modelId);

  if (!model) {
    return;
  }

  await model.highlight(
    [node.localId],
    createHighlight(0xff00ff, "valve-dropdown-selected"),
  );

  highlightedValveFromDropdown.value = node;

  await fragmentManager.core.update(true);
}

async function linkSelectedPipesToPreparedValve() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    flowMessage.value =
      "Seleciona uma válvula no dropdown ou no modelo antes de associar tubos.";
    return;
  }

  const highlightedRoute = getHighlightedRouteForValveAssociation();

  if (!highlightedRoute) {
  flowMessage.value =
    "Seleciona primeiro o caminho que esta válvula deve controlar.";
  return;
}

  if (!selectedCount.value) {
    flowMessage.value =
      "Seleciona o primeiro tubo a partir do qual a válvula deve bloquear.";
    return;
  }

  if (!flowConnections.length && savedRoutes.length && loadedModels.size) {
    await applyAllSavedRoutes();
  }

  const valveKey = nodeKey(valveNode);
  const linkedPipesByKey = new Map<string, ValveControlledPipeLink>();

  for (const [modelId, ids] of selectedItems) {
    for (const localId of ids) {
      const selectedNode: FlowNode = {
        modelId,
        localId,
      };

      if (isSameNode(selectedNode, valveNode)) {
        continue;
      }

      const downstreamPipes = findDownstreamPipesFromNode(selectedNode);

      for (const pipeNode of downstreamPipes) {
        linkedPipesByKey.set(
  `${pipeNode.routeId}|${nodeKey(pipeNode)}`,
  pipeNode,
);
      }
    }
  }

  const linkedPipes = [...linkedPipesByKey.values()].map((pipeNode) => ({
  ...pipeNode,
  switchMode: selectedValveSwitchMode.value,
  targetTemperature: getTargetCircuitForValveSwitch(
    pipeNode.temperature,
    selectedValveSwitchMode.value,
  ),
}));

  if (!linkedPipes.length) {
    flowMessage.value =
      `O tubo selecionado não pertence ao caminho realçado "${highlightedRoute.name}".`;
    return;
  }

  valveControlledPipeLinks.set(valveKey, linkedPipes);
  saveValvePipeLinksToStorage();

  const valveElement = mepElements[elementKey(
    valveNode.modelId,
    valveNode.localId,
  )];

  if (valveElement?.state === "closed") {
    for (const pipeNode of linkedPipes) {
      blockPipeForRoute(pipeNode.routeId, pipeNode);
    }

    valveBlockedPipeLinks.set(valveKey, linkedPipes);
  }

  updateBlockedCount();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value =
  `Válvula associada ao caminho "${highlightedRoute.name}" ` +
  `a partir do tubo #${linkedPipes[0]?.localId}. ` +
  `${linkedPipes.length} tubo(s) serão controlados. ` +
  `Troca: ${getValveSwitchModeLabel(selectedValveSwitchMode.value)}.`;

}

async function removeSelectedValvePipeLink() {
  const valveNode = getValveNodeForControl();

  if (!valveNode) {
    flowMessage.value =
      "Seleciona uma válvula no dropdown ou no modelo para remover a associação.";
    return;
  }

  const valveKey = nodeKey(valveNode);

  const linkedPipes =
    valveControlledPipeLinks.get(valveKey) ??
    valveBlockedPipeLinks.get(valveKey) ??
    [];

  if (!linkedPipes.length) {
    flowMessage.value =
      "A válvula selecionada não tem associação guardada.";
    return;
  }

  const shouldRemove = confirm(
  "Tens a certeza que queres remover a associação desta válvula?",
);

if (!shouldRemove) {
  flowMessage.value =
    "Remoção da associação cancelada.";
  return;
}

restoreValveLinkedPipesToOriginalCircuit(linkedPipes);

  for (const pipeNode of linkedPipes) {
    unblockPipeForRoute(pipeNode.routeId, pipeNode);
  }

  valveControlledPipeLinks.delete(valveKey);
  valveBlockedPipeLinks.delete(valveKey);

  saveValvePipeLinksToStorage();
  updateBlockedCount();

  await rebuildManualFlowLayer();

  flowMessage.value =
    "Associação da válvula removida.";
}

async function setSelectedValvesState(state: "open" | "closed") {
  const valveNodes = getValveNodesForControl();

  if (!valveNodes.length) {
    flowMessage.value = "Seleciona primeiro uma válvula.";
    return;
  }

  let changedCount = 0;

  for (const valveNode of valveNodes) {
    const { modelId, localId } = valveNode;

    if (!isIsolationValve(modelId, localId)) {
      continue;
    }

    const key = elementKey(modelId, localId);
    const existingElement = mepElements[key];

    if (!existingElement || !isValveElementType(existingElement.elementType)) {
      continue;
    }

    mepElements[key] = {
      ...existingElement,
      modelId,
      localId,
      state,
    };

    const valveKey = nodeKey(valveNode);
    const linkedPipes = getLinkedPipesForValveNode(valveNode);

    let blockedSet = blockedPipes.get(modelId);

    if (state === "closed") {
      restoreValveLinkedPipesToOriginalCircuit(linkedPipes);

      if (!blockedSet) {
        blockedSet = new Set<number>();
        blockedPipes.set(modelId, blockedSet);
      }

      blockedSet.add(localId);

      for (const pipeNode of linkedPipes) {
        blockPipeForRoute(pipeNode.routeId, pipeNode);
      }

      valveBlockedPipeLinks.set(valveKey, linkedPipes);
    } else {
      blockedSet?.delete(localId);

      if (blockedSet && blockedSet.size === 0) {
        blockedPipes.delete(modelId);
      }

      for (const pipeNode of linkedPipes) {
        unblockPipeForRoute(pipeNode.routeId, pipeNode);
      }

      valveBlockedPipeLinks.delete(valveKey);

      if (existingElement.elementType === "normallyClosedValve") {
        applyValveSwitchToLinkedPipes(linkedPipes);
      } else {
        restoreValveLinkedPipesToOriginalCircuit(linkedPipes);
      }
    }

    changedCount++;
  }

  if (!changedCount) {
    flowMessage.value = "Nenhuma válvula válida encontrada para alterar.";
    return;
  }

  updateBlockedCount();
  updateManualStats();
  saveMepElementsToStorage();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  await showSelectedMepElementInfo();

  flowMessage.value =
    state === "closed"
      ? `${changedCount} válvula(s) fechada(s). Fluxo bloqueado a jusante.`
      : `${changedCount} válvula(s) aberta(s). Fluxo reposto a jusante.`;
}

async function applyInverseNormalStateToSelectedValves() {
  const selectedValves = getSelectedValveElements();

  if (!selectedValves.length) {
    flowMessage.value =
      "Seleciona uma válvula no dropdown ou no modelo antes de alternar.";

    return;
  }

  const targetState = getInverseValveState(selectedValves[0].elementType);

  const hasMixedInverseStates = selectedValves.some(
    (valve) => getInverseValveState(valve.elementType) !== targetState,
  );

  if (hasMixedInverseStates) {
    flowMessage.value =
      "Seleciona válvulas com o mesmo estado normal para usar esta ação.";
    return;
  }

  await setSelectedValvesState(targetState);
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

  const lockedRouteNames = selectedItemsHaveLockedRouteNodes();

if (lockedRouteNames.length) {
  flowMessage.value =
    `Não é possível sincronizar/inverter o sentido. ` +
    `A seleção contém tubo(s) de caminho(s) protegido(s): ` +
    lockedRouteNames.join(", ") +
    `. Desprotege primeiro o caminho.`;

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

async function resetSelectedValvesToNormal() {
  const valveNodes = getValveNodesForControl();

  if (!valveNodes.length) {
    flowMessage.value = "Seleciona primeiro uma válvula.";
    return;
  }

  let changedCount = 0;

  for (const valveNode of valveNodes) {
    const { modelId, localId } = valveNode;
    const key = elementKey(modelId, localId);
    const element = mepElements[key];

    if (!element || !isValveElementType(element.elementType)) {
      continue;
    }

    const normalState = getNormalValveState(element.elementType);

    mepElements[key] = {
      ...element,
      state: normalState,
    };

    const valveKey = nodeKey(valveNode);
    const linkedPipes = getLinkedPipesForValveNode(valveNode);

    restoreValveLinkedPipesToOriginalCircuit(linkedPipes);

    let blockedSet = blockedPipes.get(modelId);

    if (normalState === "closed") {
      if (!blockedSet) {
        blockedSet = new Set<number>();
        blockedPipes.set(modelId, blockedSet);
      }

      blockedSet.add(localId);

      for (const pipeNode of linkedPipes) {
        blockPipeForRoute(pipeNode.routeId, pipeNode);
      }

      valveBlockedPipeLinks.set(valveKey, linkedPipes);
    } else {
      blockedSet?.delete(localId);

      if (blockedSet && blockedSet.size === 0) {
        blockedPipes.delete(modelId);
      }

      for (const pipeNode of linkedPipes) {
        unblockPipeForRoute(pipeNode.routeId, pipeNode);
      }

      valveBlockedPipeLinks.delete(valveKey);
    }

    changedCount++;
  }

  if (!changedCount) {
    flowMessage.value = "Nenhuma válvula válida encontrada para repor.";
    return;
  }

  updateBlockedCount();
  updateManualStats();
  saveMepElementsToStorage();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  await showSelectedMepElementInfo();

  flowMessage.value = `${changedCount} válvula(s) reposta(s) ao estado normal.`;
}

async function clearBlockedPipes() {
  blockedPipes.clear();
  blockedRoutePipes.clear();
  valveBlockedPipeLinks.clear();

  if (!flowConnections.length && savedRoutes.length && loadedModels.size) {
    await applyAllSavedRoutes();
  }

  for (const key of Object.keys(mepElements)) {
    const element = mepElements[key];

    if (!isValveElementType(element.elementType)) {
      continue;
    }

    const normalState = getNormalValveState(element.elementType);

    mepElements[key] = {
      ...element,
      state: normalState,
    };

    const valveNode: FlowNode = {
  modelId: element.modelId,
  localId: element.localId,
};

const valveKey = nodeKey(valveNode);
const linkedPipes = valveControlledPipeLinks.get(valveKey) ?? [];

restoreValveLinkedPipesToOriginalCircuit(linkedPipes);

if (normalState === "closed") {
  for (const pipeNode of linkedPipes) {
    blockPipeForRoute(pipeNode.routeId, pipeNode);
  }

  valveBlockedPipeLinks.set(valveKey, linkedPipes);
}
  }

  updateBlockedCount();

  await rebuildManualFlowLayer();

  saveMepElementsToStorage();

  flowMessage.value = "Válvulas repostas ao estado normal.";
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

function isSameConnection(
  firstConnection: FlowConnection,
  secondConnection: FlowConnection,
) {
  return (
    firstConnection.temperature === secondConnection.temperature &&
    isSameNode(firstConnection.from, secondConnection.from) &&
    isSameNode(firstConnection.to, secondConnection.to)
  );
}

function addFlowConnectionIfMissing(connection: FlowConnection) {
  const alreadyExists = flowConnections.some(
    (existingConnection) =>
      existingConnection.temperature === connection.temperature &&
      isSameNode(existingConnection.from, connection.from) &&
      isSameNode(existingConnection.to, connection.to),
  );

  if (!alreadyExists) {
    flowConnections.push(connection);
  }
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

function getSupplyCircuitKey(cycleNumber: number) {
  return `supply${cycleNumber}`;
}

function getReturnCircuitKey(cycleNumber: number) {
  return `return${cycleNumber}`;
}

function getAvailableCircuits() {
  ensureCycleCircuitDefinitions();

  return cycleCircuitDefinitions.map((circuit) => circuit.key);
}

function getAllKnownCircuitKeys() {
  return [
    ...new Set([
      ...getAvailableCircuits(),
      ...Object.keys(manualAssignments),
      ...savedRoutes.map((route) => route.temperature),
    ]),
  ];
}

function getCircuitCycleNumber(circuit: PipeCircuit) {
  const definition = getCycleCircuitDefinition(circuit);

  if (definition) {
    return definition.cycleNumber;
  }

  const match = circuit.match(/\d+/);
  return match ? Number(match[0]) : 1;
}

function isSupplyCircuit(circuit: PipeCircuit) {
  const definition = getCycleCircuitDefinition(circuit);

  if (definition) {
    return (
      definition.kind === "hotSupply" ||
      definition.kind === "coldSupply"
    );
  }

  return circuit.startsWith("supply");
}

function isReturnCircuit(circuit: PipeCircuit) {
  const definition = getCycleCircuitDefinition(circuit);

  if (definition) {
    return (
      definition.kind === "hotReturn" ||
      definition.kind === "coldReturn"
    );
  }

  return circuit.startsWith("return");
}

function getCircuitLabel(circuit: PipeCircuit) {
  const definition = getCycleCircuitDefinition(circuit);

  if (definition) {
    return `${definition.name} ${getCycleDisplayName(definition.cycleNumber)}`;
  }

  const cycleNumber = getCircuitCycleNumber(circuit);
  const cycleName = getCycleDisplayName(cycleNumber);

  if (isSupplyCircuit(circuit)) {
    return `ida ${cycleName}`;
  }

  if (isReturnCircuit(circuit)) {
    return `retorno ${cycleName}`;
  }

  return circuit;
}

function getPipeStat(circuit: PipeCircuit) {
  return pipeStats[circuit] ?? 0;
}

function ensureCycleNames() {
  for (let cycleNumber = 1; cycleNumber <= waterCycleCount.value; cycleNumber++) {
    const key = String(cycleNumber);

    if (!cycleNames[key]) {
      cycleNames[key] = String(cycleNumber);
    }
  }

  for (const key of Object.keys(cycleNames)) {
    const cycleNumber = Number(key);

    if (cycleNumber > waterCycleCount.value) {
      delete cycleNames[key];
    }
  }
}

function getDefaultSupplyColor(cycleNumber: number) {
  const supplyColors = [
    "#ff0000",
    "#ff5252",
    "#ff8a80",
    "#d50000",
    "#ff1744",
    "#b71c1c",
  ];

  return supplyColors[(cycleNumber - 1) % supplyColors.length];
}

function getDefaultReturnColor(cycleNumber: number) {
  const returnColors = [
    "#ff8c00",
    "#ffa726",
    "#ffc107",
    "#ff6d00",
    "#ffb300",
    "#e65100",
  ];

  return returnColors[(cycleNumber - 1) % returnColors.length];
}

function getDefaultColdSupplyColor(cycleNumber: number) {
  const coldSupplyColors = [
    "#0077ff",
    "#2196f3",
    "#03a9f4",
    "#00bcd4",
    "#1565c0",
    "#4fc3f7",
  ];

  return coldSupplyColors[(cycleNumber - 1) % coldSupplyColors.length];
}

function getDefaultColdReturnColor(cycleNumber: number) {
  const coldReturnColors = [
    "#7b1fa2",
    "#9c27b0",
    "#ba68c8",
    "#673ab7",
    "#512da8",
    "#9575cd",
  ];

  return coldReturnColors[(cycleNumber - 1) % coldReturnColors.length];
}

function getDefaultExtraColor(cycleNumber: number) {
  const extraColors = [
    "#2e7d32",
    "#43a047",
    "#66bb6a",
    "#1b5e20",
    "#81c784",
    "#558b2f",
  ];

  return extraColors[(cycleNumber - 1) % extraColors.length];
}

function getNextCircuitNameForKind(
  kind: CycleCircuitKind,
  cycleNumber: number,
) {
  const existingSameKindCount = cycleCircuitDefinitions.filter(
    (circuit) =>
      circuit.cycleNumber === cycleNumber &&
      circuit.kind === kind,
  ).length;

  const nextNumber = existingSameKindCount + 1;

  if (kind === "hotSupply") {
    return `Ida quente ${nextNumber}`;
  }

  if (kind === "coldSupply") {
    return `Ida fria ${nextNumber}`;
  }

  if (kind === "hotReturn") {
    return `Retorno quente ${nextNumber}`;
  }

  if (kind === "coldReturn") {
    return `Retorno frio ${nextNumber}`;
  }

  return `Extra ${nextNumber}`;
}

function getDefaultCircuitColor(
  kind: CycleCircuitKind,
  cycleNumber: number,
) {
  if (kind === "hotSupply") {
    return getDefaultSupplyColor(cycleNumber);
  }

  if (kind === "coldSupply") {
    return getDefaultColdSupplyColor(cycleNumber);
  }

  if (kind === "hotReturn") {
    return getDefaultReturnColor(cycleNumber);
  }

  if (kind === "coldReturn") {
    return getDefaultColdReturnColor(cycleNumber);
  }

  return getDefaultExtraColor(cycleNumber);
}

function getNextCircuitColorForKind(kind: CycleCircuitKind) {
  const existingSameKindCount = cycleCircuitDefinitions.filter(
    (circuit) => circuit.kind === kind,
  ).length;

  if (kind === "hotSupply") {
    const hotSupplyColors = [
      "#ff0000",
      "#ff5252",
      "#ff8a80",
      "#d50000",
      "#ff1744",
      "#b71c1c",
    ];

    return hotSupplyColors[existingSameKindCount % hotSupplyColors.length];
  }

  if (kind === "coldSupply") {
    const coldSupplyColors = [
      "#0077ff",
      "#2196f3",
      "#03a9f4",
      "#00bcd4",
      "#1565c0",
      "#4fc3f7",
    ];

    return coldSupplyColors[existingSameKindCount % coldSupplyColors.length];
  }

  if (kind === "hotReturn") {
    const hotReturnColors = [
      "#ff8c00",
      "#ffa726",
      "#ffc107",
      "#ff6d00",
      "#ffb300",
      "#e65100",
    ];

    return hotReturnColors[existingSameKindCount % hotReturnColors.length];
  }

  if (kind === "coldReturn") {
    const coldReturnColors = [
      "#7b1fa2",
      "#9c27b0",
      "#ba68c8",
      "#673ab7",
      "#512da8",
      "#9575cd",
    ];

    return coldReturnColors[existingSameKindCount % coldReturnColors.length];
  }

  const extraColors = [
    "#2e7d32",
    "#43a047",
    "#66bb6a",
    "#1b5e20",
    "#81c784",
    "#558b2f",
  ];

  return extraColors[existingSameKindCount % extraColors.length];
}

function updatePendingCycleCircuitColorFromKind() {
  pendingCycleCircuitColor.value = getNextCircuitColorForKind(
    pendingCycleCircuitKind.value,
  );
}

function getDefaultCircuitKey(
  cycleNumber: number,
  kind: CycleCircuitKind,
) {
  return `cycle${cycleNumber}-${kind}-${crypto.randomUUID()}`;
}

function hasCycleCircuitUsage(circuitKey: PipeCircuit) {
  const hasAssignments =
    [...(manualAssignments[circuitKey]?.values() ?? [])].some(
      (ids) => ids.size > 0,
    );

  const hasRoutes = savedRoutes.some(
    (route) => route.temperature === circuitKey,
  );

  const hasFlowConnections = flowConnections.some(
    (connection) => connection.temperature === circuitKey,
  );

  return hasAssignments || hasRoutes || hasFlowConnections;
}

function removeUnusedLegacyDefaultCycleCircuits() {
  for (let index = cycleCircuitDefinitions.length - 1; index >= 0; index--) {
    const circuit = cycleCircuitDefinitions[index];

    if (!circuit.lockedDefault) {
      continue;
    }

    if (hasCycleCircuitUsage(circuit.key)) {
      cycleCircuitDefinitions[index] = {
        ...circuit,
        lockedDefault: false,
      };
      continue;
    }

    cycleCircuitDefinitions.splice(index, 1);
  }
}

function ensureCycleCircuitDefinitions() {
  removeUnusedLegacyDefaultCycleCircuits();

  for (let index = cycleCircuitDefinitions.length - 1; index >= 0; index--) {
    if (cycleCircuitDefinitions[index].cycleNumber > waterCycleCount.value) {
      cycleCircuitDefinitions.splice(index, 1);
    }
  }

  if (activeCycleNumber.value > waterCycleCount.value) {
    activeCycleNumber.value = waterCycleCount.value;
  }

  if (activeCycleNumber.value < 1) {
    activeCycleNumber.value = 1;
  }

  selectDefaultCircuitForActiveCycle();
}

function saveCycleCircuitDefinitionsToStorage() {
  localStorage.setItem(
    CYCLE_CIRCUITS_STORAGE_KEY,
    JSON.stringify(cycleCircuitDefinitions),
  );
}

function loadCycleCircuitDefinitionsFromStorage() {
  const saved = localStorage.getItem(CYCLE_CIRCUITS_STORAGE_KEY);

  if (saved) {
    try {
      const parsed = JSON.parse(saved) as CycleCircuitDefinition[];
      cycleCircuitDefinitions.splice(0);
      cycleCircuitDefinitions.push(...parsed);
    } catch (error) {
      console.error("Erro ao carregar circuitos dos ciclos:", error);
    }
  }

  ensureCycleCircuitDefinitions();
}

function getCycleCircuitDefinition(circuitKey: PipeCircuit) {
  return cycleCircuitDefinitions.find(
    (circuit) => circuit.key === circuitKey,
  );
}

function getCycleCircuitDefinitionsForCycle(cycleNumber: number) {
  return cycleCircuitDefinitions.filter(
    (circuit) => circuit.cycleNumber === cycleNumber,
  );
}

function getActiveCycleCircuitDefinitions() {
  return getCycleCircuitDefinitionsForCycle(activeCycleNumber.value);
}

function getCycleLegendDisplayName(cycleNumber: number) {
  const cycleName = getCycleDisplayName(cycleNumber);

  if (cycleName === String(cycleNumber)) {
    return `Ciclo ${cycleNumber}`;
  }

  return cycleName;
}

function getCycleCircuitLegendGroups() {
  const groups = [];

  for (
    let cycleNumber = 1;
    cycleNumber <= waterCycleCount.value;
    cycleNumber++
  ) {
    const circuits = getCycleCircuitDefinitionsForCycle(cycleNumber);

    if (!circuits.length) {
      continue;
    }

    groups.push({
      cycleNumber,
      cycleName: getCycleLegendDisplayName(cycleNumber),
      circuits: [...circuits].sort((firstCircuit, secondCircuit) =>
        firstCircuit.name.localeCompare(secondCircuit.name),
      ),
    });
  }

  return groups;
}

function getAllCycleCircuitDefinitionsForLegend() {
  return [...cycleCircuitDefinitions].sort((firstCircuit, secondCircuit) => {
    if (firstCircuit.cycleNumber !== secondCircuit.cycleNumber) {
      return firstCircuit.cycleNumber - secondCircuit.cycleNumber;
    }

    return firstCircuit.name.localeCompare(secondCircuit.name);
  });
}

function selectDefaultCircuitForActiveCycle() {
  const activeCircuits = getActiveCycleCircuitDefinitions();

  if (!activeCircuits.length) {
    selectedCycleCircuitKey.value = "";
    return;
  }

  const stillValid = activeCircuits.some(
    (circuit) => circuit.key === selectedCycleCircuitKey.value,
  );

  if (!stillValid) {
    selectedCycleCircuitKey.value = activeCircuits[0].key;
  }
}

function getSelectedCycleCircuitDefinition() {
  selectDefaultCircuitForActiveCycle();

  return (
    cycleCircuitDefinitions.find(
      (circuit) => circuit.key === selectedCycleCircuitKey.value,
    ) ?? null
  );
}

async function createAutoRouteForSelectedCycleCircuit() {
  const circuit = getSelectedCycleCircuitDefinition();

  if (!circuit) {
    flowMessage.value = "Seleciona primeiro um tipo de caminho do ciclo.";
    return;
  }

  await createAutoRoute(circuit.key);
}

async function createManualRouteForSelectedCycleCircuit() {
  const circuit = getSelectedCycleCircuitDefinition();

  if (!circuit) {
    flowMessage.value = "Seleciona primeiro um tipo de caminho do ciclo.";
    return;
  }

  await createManualRouteFromSelection(circuit.key);
}

function getCycleCircuitDefinitionsByKind(
  cycleNumber: number,
  kind: CycleCircuitKind,
) {
  return cycleCircuitDefinitions.filter(
    (circuit) =>
      circuit.cycleNumber === cycleNumber &&
      circuit.kind === kind,
  );
}

function getCycleCircuitKindLabel(kind: CycleCircuitKind) {
  if (kind === "hotSupply") {
    return "ida quente";
  }

  if (kind === "coldSupply") {
    return "ida fria";
  }

  if (kind === "hotReturn") {
    return "retorno quente";
  }

  if (kind === "coldReturn") {
    return "retorno frio";
  }

  return "extra";
}

function toggleCycleCircuitPanel() {
  isCycleCircuitPanelOpen.value = !isCycleCircuitPanelOpen.value;

  if (isCycleCircuitPanelOpen.value) {
    updatePendingCycleCircuitColorFromKind();
  }
}

function createCycleCircuitDefinition() {
  const cycleNumber = activeCycleNumber.value;
  const kind = pendingCycleCircuitKind.value;
  const trimmedName = pendingCycleCircuitName.value.trim();

const name =
  trimmedName ||
  getNextCircuitNameForKind(kind, cycleNumber);

  const key = `cycle${cycleNumber}-${kind}-${crypto.randomUUID()}`;

  const selectedColor =
  pendingCycleCircuitColor.value ||
  getNextCircuitColorForKind(kind);

cycleCircuitDefinitions.push({
  key,
  cycleNumber,
  kind,
  name,
  color: selectedColor,
  defaultColor: selectedColor,
});

  selectedCycleCircuitKey.value = key;

  if (!manualAssignments[key]) {
    manualAssignments[key] = new Map();
  }

  saveCycleCircuitDefinitionsToStorage();

  pendingCycleCircuitName.value = "";
pendingCycleCircuitColor.value = getNextCircuitColorForKind(kind);

  flowMessage.value =
    `${capitalizeFirstLetter(getCycleCircuitKindLabel(kind))} "${name}" criado no ciclo ${getCycleDisplayName(cycleNumber)}.`;
}

async function deleteCycleCircuitDefinition(circuitKey: PipeCircuit) {
  const circuit = getCycleCircuitDefinition(circuitKey);

  if (!circuit) {
    return;
  }

  if (circuit.locked) {
  flowMessage.value =
    "O caminho \"" +
    circuit.name +
    "\" está protegido. Desprotege primeiro para apagar.";
  return;
}

  const hasAssignments =
    [...(manualAssignments[circuitKey]?.values() ?? [])].some(
      (ids) => ids.size > 0,
    );

  const hasRoutes = savedRoutes.some(
    (route) => route.temperature === circuitKey,
  );

  const hasFlowConnections = flowConnections.some(
    (connection) => connection.temperature === circuitKey,
  );

  const hasUsage = hasAssignments || hasRoutes || hasFlowConnections;

  const shouldDelete = confirm(
    hasUsage
      ? "O caminho \"" +
          circuit.name +
          "\" tem marcações, ligações ou caminhos guardados. Queres apagar mesmo assim?"
      : "Tens a certeza que queres apagar o caminho \"" +
          circuit.name +
          "\"?",
  );

  if (!shouldDelete) {
    flowMessage.value = "Remoção do caminho cancelada.";
    return;
  }

  delete manualAssignments[circuitKey];

  for (let index = savedRoutes.length - 1; index >= 0; index--) {
    if (savedRoutes[index].temperature === circuitKey) {
      savedRoutes.splice(index, 1);
    }
  }

  for (let index = flowConnections.length - 1; index >= 0; index--) {
    if (flowConnections[index].temperature === circuitKey) {
      flowConnections.splice(index, 1);
    }
  }

  const definitionIndex = cycleCircuitDefinitions.findIndex(
    (item) => item.key === circuitKey,
  );

  if (definitionIndex !== -1) {
    cycleCircuitDefinitions.splice(definitionIndex, 1);
  }

  if (selectedCycleCircuitKey.value === circuitKey) {
    selectedCycleCircuitKey.value = "";
    selectDefaultCircuitForActiveCycle();
  }

  saveCycleCircuitDefinitionsToStorage();
  saveRoutesToStorage();
  updateManualStats();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  } else {
    clearFlowVisuals();
    await fragmentManager.core.update(true);
  }

  flowMessage.value =
    "Caminho \"" +
    circuit.name +
    "\" apagado.";
}

async function saveCycleCircuitDefinitionChanges() {
  saveCycleCircuitDefinitionsToStorage();
  circuitMaterialCache.clear();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value = "Nomes e cores dos caminhos do ciclo guardados.";
}

function getCycleCircuitDefaultColor(circuit: CycleCircuitDefinition) {
  return (
    circuit.defaultColor ||
    getDefaultCircuitColor(circuit.kind, circuit.cycleNumber)
  );
}

function isCycleCircuitColorChanged(circuit: CycleCircuitDefinition) {
  return circuit.color.toLowerCase() !==
    getCycleCircuitDefaultColor(circuit).toLowerCase();
}

function toggleCycleCircuitLock(circuitKey: PipeCircuit) {
  const circuit = getCycleCircuitDefinition(circuitKey);

  if (!circuit) {
    return;
  }

  circuit.locked = !circuit.locked;

  saveCycleCircuitDefinitionsToStorage();

  flowMessage.value = circuit.locked
    ? "Caminho \"" + circuit.name + "\" protegido."
    : "Caminho \"" + circuit.name + "\" desprotegido.";
}

function resetCycleCircuitColor(circuitKey: PipeCircuit) {
  const circuit = getCycleCircuitDefinition(circuitKey);

  if (!circuit) {
    return;
  }

  circuit.color = getCycleCircuitDefaultColor(circuit);

  saveCycleCircuitDefinitionsToStorage();
  circuitMaterialCache.clear();

  void rebuildManualFlowLayer();

  flowMessage.value = "Cor de \"" + circuit.name + "\" reposta.";
}

function getCycleDisplayName(cycleNumber: number) {
  const name = cycleNames[String(cycleNumber)]?.trim();

  return name || String(cycleNumber);
}

function saveCycleNamesToStorage() {
  localStorage.setItem(
    WATER_CYCLE_NAMES_STORAGE_KEY,
    JSON.stringify(cycleNames),
  );
}

function loadCycleNamesFromStorage() {
  const saved = localStorage.getItem(WATER_CYCLE_NAMES_STORAGE_KEY);

  if (saved) {
    try {
      const parsed = JSON.parse(saved) as Record<string, string>;

      for (const [key, value] of Object.entries(parsed)) {
        cycleNames[key] = value;
      }
    } catch (error) {
      console.error("Erro ao carregar nomes dos ciclos:", error);
    }
  }

  ensureCycleNames();
}

function toggleCycleNamesPanel() {
  isCycleNamesPanelOpen.value = !isCycleNamesPanelOpen.value;
}

function toggleSavedRoutesPanel() {
  isSavedRoutesPanelOpen.value = !isSavedRoutesPanelOpen.value;
}

function toggleCentralSummary() {
  isCentralSummaryOpen.value = !isCentralSummaryOpen.value;
}

function toggleValveDesignationPanel() {
  isValveDesignationPanelOpen.value = !isValveDesignationPanelOpen.value;
}

function toggleValveRenamePanel() {
  isValveRenamePanelOpen.value = !isValveRenamePanelOpen.value;
}

function toggleValveAssociationDetails() {
  isValveAssociationDetailsOpen.value = !isValveAssociationDetailsOpen.value;
}

function saveCycleNames() {
  ensureCycleNames();
  saveCycleNamesToStorage();

  flowMessage.value = "Nomes dos ciclos guardados.";
}

function getCircuitColorStyle(circuit: PipeCircuit) {
  return `#${getCircuitColor(circuit).toString(16).padStart(6, "0")}`;
}

function saveWaterCycleCountToStorage() {
  localStorage.setItem(
    WATER_CYCLE_COUNT_STORAGE_KEY,
    String(waterCycleCount.value),
  );
}

function loadWaterCycleCountFromStorage() {
  const saved = localStorage.getItem(WATER_CYCLE_COUNT_STORAGE_KEY);
  const parsed = saved ? Number(saved) : 3;

  const safeValue = Number.isFinite(parsed)
    ? Math.max(1, Math.min(12, Math.round(parsed)))
    : 3;

  waterCycleCount.value = safeValue;
  pendingWaterCycleCount.value = safeValue;

  ensureConfiguredAssignments();
}

function ensureConfiguredAssignments() {
  for (const circuit of getAvailableCircuits()) {
    if (!manualAssignments[circuit]) {
      manualAssignments[circuit] = new Map();
    }
  }
}

async function applyWaterCycleCount() {
  const nextCount = Math.max(
    1,
    Math.min(12, Math.round(Number(pendingWaterCycleCount.value) || 1)),
  );

  const previousCount = waterCycleCount.value;

  if (nextCount === previousCount) {
    flowMessage.value = `A central já está configurada com ${nextCount} ciclo(s) de água.`;
    return;
  }

  if (nextCount < previousCount) {
    const removedCircuits: PipeCircuit[] = [];

    for (
      let cycleNumber = nextCount + 1;
      cycleNumber <= previousCount;
      cycleNumber++
    ) {
      removedCircuits.push(getSupplyCircuitKey(cycleNumber));
      removedCircuits.push(getReturnCircuitKey(cycleNumber));
    }

    const hasAssignmentsToRemove = removedCircuits.some(
      (circuit) =>
        [...(manualAssignments[circuit]?.values() ?? [])].some(
          (ids) => ids.size > 0,
        ),
    );

    const hasRoutesToRemove = savedRoutes.some((route) =>
      removedCircuits.includes(route.temperature),
    );

    if (hasAssignmentsToRemove || hasRoutesToRemove) {
      const shouldContinue = confirm(
        `Existem marcações ou caminhos nos ciclos que vão ser removidos. ` +
          `Deseja continuar e apagar esses dados?`,
      );

      if (!shouldContinue) {
        pendingWaterCycleCount.value = previousCount;
        flowMessage.value = "Alteração do número de ciclos cancelada.";
        return;
      }
    }

    for (const circuit of removedCircuits) {
      delete manualAssignments[circuit];
    }

    for (let index = savedRoutes.length - 1; index >= 0; index--) {
      if (removedCircuits.includes(savedRoutes[index].temperature)) {
        savedRoutes.splice(index, 1);
      }
    }

    for (let index = flowConnections.length - 1; index >= 0; index--) {
      if (removedCircuits.includes(flowConnections[index].temperature)) {
        flowConnections.splice(index, 1);
      }
    }

    for (let index = currentRouteConnections.length - 1; index >= 0; index--) {
      if (removedCircuits.includes(currentRouteConnections[index].temperature)) {
        currentRouteConnections.splice(index, 1);
      }
    }

    saveRoutesToStorage();
  }

  waterCycleCount.value = nextCount;
pendingWaterCycleCount.value = nextCount;

if (activeCycleNumber.value > nextCount) {
  activeCycleNumber.value = nextCount;
}

if (activeCycleNumber.value < 1) {
  activeCycleNumber.value = 1;
}

ensureConfiguredAssignments();
ensureCycleNames();
ensureCycleCircuitDefinitions();
saveWaterCycleCountToStorage();
saveCycleNamesToStorage();
saveCycleCircuitDefinitionsToStorage();
updateManualStats();

  if (countAssignments() > 0 || flowConnections.length > 0) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value =
    `Número de ciclos de água atualizado para ${nextCount}.`;
}

function capitalizeFirstLetter(text: string) {
  return text.charAt(0).toUpperCase() + text.slice(1);
}

function getRoutePipeCount(route: SavedRoute) {
  return route.path.length;
}

function getRouteVisibilityLabel(route: SavedRoute) {
  return route.hidden ? "oculto" : "visível";
}

function getRouteProtectionLabel(route: SavedRoute) {
  return route.locked ? "protegido" : "editável";
}

function isSavedRouteBlocked(route: SavedRoute) {
  return route.path.some((node) =>
    blockedRoutePipes.has(routeBlockedPipeKey(route.id, node)),
  );
}

function getValveLabelForBlockedRoute(route: SavedRoute) {
  const hasAssociatedValve = [...valveControlledPipeLinks.values()].some(
    (linkedPipes) =>
      linkedPipes.some((pipeNode) => pipeNode.routeId === route.id),
  );

  return hasAssociatedValve ? "bloqueado por válvula" : "";
}

function getRouteBlockedLabel(route: SavedRoute) {
  return isSavedRouteBlocked(route) ? "bloqueado" : "ativo";
}

function getRouteCircuitDisplayLabel(route: SavedRoute) {
  return capitalizeFirstLetter(getCircuitLabel(route.temperature));
}

function getValveSwitchModeLabel(switchMode: ValveSwitchMode) {
  if (switchMode === "switchToSupply") {
    return "trocar para ida";
  }

  if (switchMode === "switchToReturn") {
    return "trocar para retorno";
  }

  return "não trocar";
}

function getTargetCircuitForValveSwitch(
  sourceCircuit: PipeCircuit,
  switchMode: ValveSwitchMode,
) {
  const cycleNumber = getCircuitCycleNumber(sourceCircuit);

  if (switchMode === "switchToSupply") {
    return getSupplyCircuitKey(cycleNumber);
  }

  if (switchMode === "switchToReturn") {
    return getReturnCircuitKey(cycleNumber);
  }

  return sourceCircuit;
}

function assignNodeToOnlyOneCircuit(
  node: FlowNode,
  circuit: PipeCircuit,
) {
  for (const existingCircuit of getAllKnownCircuitKeys()) {
    getAssignmentSet(existingCircuit, node.modelId).delete(node.localId);
  }

  getAssignmentSet(circuit, node.modelId).add(node.localId);
}

function isNodeSharedWithOtherVisibleRoute(
  routeId: string,
  node: FlowNode,
) {
  return savedRoutes.some(
    (route) =>
      route.id !== routeId &&
      !route.hidden &&
      routeContainsAdaptedNode(route, node),
  );
}

function restoreValveLinkedPipesToOriginalCircuit(
  linkedPipes: ValveControlledPipeLink[],
) {
  for (const pipeNode of linkedPipes) {
    if (isNodeSharedWithOtherVisibleRoute(pipeNode.routeId, pipeNode)) {
      continue;
    }

    assignNodeToOnlyOneCircuit(pipeNode, pipeNode.temperature);
  }
}

function applyValveSwitchToLinkedPipes(
  linkedPipes: ValveControlledPipeLink[],
) {
  for (const pipeNode of linkedPipes) {
    if (isNodeSharedWithOtherVisibleRoute(pipeNode.routeId, pipeNode)) {
      continue;
    }

    const switchMode = pipeNode.switchMode ?? "none";

    if (switchMode === "none") {
      assignNodeToOnlyOneCircuit(pipeNode, pipeNode.temperature);
      continue;
    }

    const targetTemperature =
      pipeNode.targetTemperature ??
      getTargetCircuitForValveSwitch(
        pipeNode.temperature,
        switchMode,
      );

    assignNodeToOnlyOneCircuit(pipeNode, targetTemperature);
  }
}

function routeContainsNode(route: SavedRoute, node: FlowNode) {
  return route.path.some((routeNode) => isSameNode(routeNode, node));
}

function getLockedRoutesForNode(node: FlowNode) {
  return savedRoutes.filter(
    (route) =>
      route.locked &&
      routeContainsNode(route, node),
  );
}

function selectedItemsHaveLockedRouteNodes() {
  const lockedRouteNames = new Set<string>();

  for (const [modelId, ids] of selectedItems) {
    for (const localId of ids) {
      const node: FlowNode = {
        modelId,
        localId,
      };

      const lockedRoutes = getLockedRoutesForNode(node);

      for (const route of lockedRoutes) {
        lockedRouteNames.add(route.name);
      }
    }
  }

  return [...lockedRouteNames];
}

function isNodeHiddenBySavedRouteVisibility(node: FlowNode) {
  const routesWithNode = savedRoutes.filter((route) =>
    routeContainsAdaptedNode(route, node),
  );

  if (!routesWithNode.length) {
    return false;
  }

  const hasVisibleRouteWithNode = routesWithNode.some(
    (route) => !route.hidden,
  );

  if (hasVisibleRouteWithNode) {
    return false;
  }

  return routesWithNode.some((route) => route.hidden);
}

function getNextRouteNumberForCircuit(circuit: PipeCircuit) {
  return (
    savedRoutes.filter(
      (route) => route.temperature === circuit,
    ).length + 1
  );
}

function getNodeTemperature(node: FlowNode): PipeCircuit | null {
  for (const circuit of getAllKnownCircuitKeys()) {
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
  const material = getCircuitMaterial(temperature);
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
  color: getCircuitColor(temperature),
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
) {
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

  const material = getCircuitMaterial(temperature);

  const particleCount = Math.max(1, Math.round(length / 0.4));

  for (let i = 0; i < particleCount; i++) {
    const mesh = new THREE.Mesh(geometry, material);
    mesh.renderOrder = 20;

    mesh.quaternion.setFromUnitVectors(
      new THREE.Vector3(0, 1, 0),
      direction,
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

function getPointToSegmentDistance(point: any, segmentStart: any, segmentEnd: any) {
  const segment = segmentEnd.clone().sub(segmentStart);
  const pointVector = point.clone().sub(segmentStart);

  const segmentLengthSquared = segment.lengthSq();

  if (segmentLengthSquared === 0) {
    return point.distanceTo(segmentStart);
  }

  const t = Math.max(
    0,
    Math.min(1, pointVector.dot(segment) / segmentLengthSquared),
  );

  const projectedPoint = segmentStart.clone().add(segment.multiplyScalar(t));

  return point.distanceTo(projectedPoint);
}

function findDownstreamPipesFromNode(startNode: FlowNode) {
  const highlightedRoute = getHighlightedRouteForValveAssociation();

  if (!highlightedRoute) {
    return [];
  }

  const loadedModelIds = [...loadedModels.keys()];
  const fallbackModelId = loadedModelIds[0];

  const adaptedPath = highlightedRoute.path.map((node) => {
    if (loadedModels.has(node.modelId)) {
      return node;
    }

    return {
      modelId: fallbackModelId,
      localId: node.localId,
    };
  });

  const startIndex = adaptedPath.findIndex((node) =>
    isSameNode(node, startNode),
  );

  if (startIndex === -1) {
    return [];
  }

  return adaptedPath.slice(startIndex).map((node) => ({
    ...node,
    temperature: highlightedRoute.temperature,
    routeId: highlightedRoute.id,
  }));
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

    const distance = getPointToSegmentDistance(
      valveCenter,
      fromCenter,
      toCenter,
    );

    if (distance < closestDistance) {
      closestDistance = distance;
      closestConnectionIndex = index;
    }
  }

  if (closestConnectionIndex === -1) {
    return [];
  }

  const firstConnection = flowConnections[closestConnectionIndex];
  const downstreamNodes: FlowNode[] = [];

  downstreamNodes.push(firstConnection.from);
  downstreamNodes.push(firstConnection.to);

  let expectedFrom = firstConnection.to;

  for (
    let index = closestConnectionIndex + 1;
    index < flowConnections.length;
    index++
  ) {
    const connection = flowConnections[index];

    if (connection.temperature !== firstConnection.temperature) {
      continue;
    }

    if (!isSameNode(connection.from, expectedFrom)) {
      break;
    }

    downstreamNodes.push(connection.to);
    expectedFrom = connection.to;
  }

  return downstreamNodes;
}

function routeBlockedPipeKey(routeId: string, node: FlowNode) {
  return `${routeId}|${nodeKey(node)}`;
}

function blockPipeForRoute(routeId: string, node: FlowNode) {
  blockedRoutePipes.add(routeBlockedPipeKey(routeId, node));
}

function unblockPipeForRoute(routeId: string, node: FlowNode) {
  blockedRoutePipes.delete(routeBlockedPipeKey(routeId, node));
}

function isPipeBlockedForRoute(routeId: string, node: FlowNode) {
  return blockedRoutePipes.has(routeBlockedPipeKey(routeId, node));
}

function routeContainsAdaptedNode(route: SavedRoute, node: FlowNode) {
  const loadedModelIds = [...loadedModels.keys()];
  const fallbackModelId = loadedModelIds[0];

  return route.path.some((routeNode) => {
    const adaptedNode = loadedModels.has(routeNode.modelId)
      ? routeNode
      : {
          modelId: fallbackModelId,
          localId: routeNode.localId,
        };

    return isSameNode(adaptedNode, node);
  });
}

function shouldHidePipeForCircuit(
  modelId: string,
  localId: number,
  circuit: PipeCircuit,
) {
  const node: FlowNode = {
    modelId,
    localId,
  };

  const visibleRoutesWithNode = savedRoutes.filter(
    (route) =>
      !route.hidden &&
      route.temperature === circuit &&
      routeContainsAdaptedNode(route, node),
  );

  if (!visibleRoutesWithNode.length) {
    return false;
  }

  return visibleRoutesWithNode.every((route) =>
    isPipeBlockedForRoute(route.id, node),
  );
}

function isPipeBlocked(modelId: string, localId: number) {
  return blockedPipes.get(modelId)?.has(localId) ?? false;
}

function isConnectionBlocked(connection: FlowConnection) {
  return false;
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

  for (const key of Object.keys(pipeStats)) {
  pipeStats[key] = 0;
}

pipeStats.supply = 0;
pipeStats.return = 0;
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
  if (!manualAssignments[circuit]) {
    manualAssignments[circuit] = new Map();
  }

  let ids = manualAssignments[circuit].get(modelId);

  if (!ids) {
    ids = new Set<number>();
    manualAssignments[circuit].set(modelId, ids);
  }

  return ids;
}

function updateManualStats() {
  let supplyTotal = 0;
  let returnTotal = 0;
  let extraTotal = 0;

  for (const circuit of getAllKnownCircuitKeys()) {
    pipeStats[circuit] = countAssignmentType(circuit);

    if (isSupplyCircuit(circuit)) {
      supplyTotal += pipeStats[circuit];
      continue;
    }

    if (isReturnCircuit(circuit)) {
      returnTotal += pipeStats[circuit];
      continue;
    }

    extraTotal += pipeStats[circuit];
  }

  pipeStats.supply = supplyTotal;
  pipeStats.return = returnTotal;
  pipeStats.total = supplyTotal + returnTotal + extraTotal;
}

function countAssignmentType(circuit: PipeCircuit) {
  return [...(manualAssignments[circuit]?.values() ?? [])].reduce(
    (total, ids) => total + ids.size,
    0,
  );
}

function countAssignments() {
  return getAllKnownCircuitKeys().reduce(
    (total, circuit) => total + countAssignmentType(circuit),
    0,
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

.saved-route-select {
  display: flex;
  align-items: center;
  gap: 6px;
}

.saved-route-select--details {
  align-items: flex-start;
}

.saved-route-text {
  display: grid;
  gap: 3px;
  line-height: 1.25;
}

.saved-route-text strong {
  color: #f7fbff;
  font-size: 0.8rem;
  font-weight: 900;
}

.saved-route-text small {
  color: #b8c9d3;
  font-size: 0.68rem;
  font-weight: 700;
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

.flow-cycle-config {
  display: grid;
  gap: 6px;
  margin-top: 12px;
  font-size: 0.82rem;
  font-weight: 800;
  color: #dbe9f1;
}

.flow-cycle-config input,
.flow-cycle-config select {
  width: 100%;
  min-height: 36px;
  border: 0;
  border-radius: 6px;
  padding: 6px 10px;
  background: #f7fbff;
  color: #111820;
  font-weight: 800;
}

.cycle-color-legend {
  display: grid;
  gap: 8px;
  margin-top: 12px;
}

.cycle-color-legend__row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

.cycle-color-legend__item {
  display: flex;
  align-items: center;
  gap: 7px;
  min-width: 0;
  padding: 7px 8px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
  color: #dbe9f1;
  font-size: 0.74rem;
  font-weight: 800;
}

.cycle-color-dot {
  flex: 0 0 auto;
  width: 12px;
  height: 12px;
  border-radius: 999px;
  box-shadow: 0 0 0 2px rgba(255, 255, 255, 0.16);
}

.cycle-name-list {
  display: grid;
  gap: 8px;
  margin-top: 12px;
}

.cycle-name-item {
  display: grid;
  grid-template-columns: 72px 1fr;
  align-items: center;
  gap: 8px;
  padding: 8px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
  color: #dbe9f1;
  font-size: 0.76rem;
  font-weight: 800;
}

.cycle-name-item input {
  width: 100%;
  min-height: 34px;
  border: 0;
  border-radius: 6px;
  padding: 6px 9px;
  background: #f7fbff;
  color: #111820;
  font-size: 0.78rem;
  font-weight: 800;
}

.saved-route-item--locked {
  border: 1px solid rgba(143, 211, 255, 0.45);
  background: rgba(143, 211, 255, 0.12);
}

.saved-route-item--locked .saved-route-text strong {
  color: #8fd3ff;
}

.route-lock-icon {
  margin-right: 4px;
}

.flow-section-title--button {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.section-collapse-button {
  display: inline-grid;
  width: 24px;
  height: 24px;
  place-items: center;
  border: 0;
  border-radius: 999px;
  background: #f7fbff;
  color: #111820;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 900;
  line-height: 1;
}

.section-collapse-button:hover {
  background: #d9f0ff;
}

.discard-route-message {
  margin: 8px 0 0;
  padding: 8px 10px;
  border-radius: 6px;
  background: rgba(255, 227, 227, 0.14);
  color: #ffd6d6;
  font-size: 0.78rem;
  font-weight: 800;
  line-height: 1.35;
}

.saved-routes-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  margin-top: 14px;
  padding: 8px 10px;
  border-radius: 6px;
  background: rgba(143, 211, 255, 0.14);
  color: #dbe9f1;
  font-size: 0.82rem;
  font-weight: 900;
}

.saved-routes-list {
  display: grid;
  gap: 6px;
  margin-top: 8px;
}

.central-summary {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 8px;
  margin-top: 12px;
}

.central-summary__item {
  display: grid;
  gap: 4px;
  min-width: 0;
  padding: 9px 10px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
}

.central-summary__item span {
  color: #b8c9d3;
  font-size: 0.68rem;
  font-weight: 800;
  line-height: 1.2;
}

.central-summary__item strong {
  color: #f7fbff;
  font-size: 1rem;
  font-weight: 900;
}

.saved-route-item--highlighted {
  border: 1px solid rgba(0, 229, 255, 0.75);
  background: rgba(0, 229, 255, 0.12);
}

.saved-route-item--highlighted .saved-route-text strong {
  color: #00e5ff;
}

.saved-route-item--blocked {
  border: 1px solid rgba(229, 57, 53, 0.85);
  background: rgba(229, 57, 53, 0.14);
}

.saved-route-item--blocked .saved-route-text strong {
  color: #ff8a80;
}

.saved-route-item--blocked .saved-route-text small {
  color: #ffd6d6;
}

.valve-switch-button {
  background: #f7fbff !important;
  color: #111820 !important;
}

.valve-switch-button--active {
  background: #8fd3ff !important;
  color: #07131a !important;
  box-shadow: 0 0 0 2px rgba(143, 211, 255, 0.35);
}

.valve-actions-layout {
  display: grid;
  grid-template-columns: 1fr;
  gap: 8px;
  margin-top: 12px;
  width: 100%;
  overflow: hidden;
}

.valve-normal-state-badge {
  width: 100%;
  padding: 7px 8px;
  border-radius: 4px;
  background: rgba(143, 211, 255, 0.14);
  border: 1px solid rgba(143, 211, 255, 0.45);
  color: #8fd3ff;
  font-size: 0.74rem;
  font-weight: 900;
  text-align: center;
  line-height: 1.2;
}

.valve-actions-buttons {
  margin-top: 0;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 6px;
  width: 100%;
}

.valve-actions-buttons button {
  min-width: 0;
  min-height: 38px;
  padding: 6px 7px;
  border-radius: 4px;
  font-size: 0.76rem;
  line-height: 1.1;
}

.valve-rename-title {
  margin-top: 12px;
  padding-top: 10px;
}

.valve-name-line {
  max-width: 100%;
  overflow-wrap: anywhere;
  line-height: 1.25;
}

.valve-details-title {
  margin-top: 10px;
  padding-top: 8px;
}

.valve-association-summary {
  display: grid;
  gap: 5px;
  margin-top: 8px;
  padding: 8px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
}

.valve-association-summary p {
  margin: 0;
  color: #dbe9f1;
  font-size: 0.76rem;
  line-height: 1.25;
}

.valve-association-summary strong {
  color: #8fd3ff;
}

.cycle-circuit-panel {
  display: grid;
  gap: 10px;
  margin-top: 10px;
}

.cycle-circuit-list {
  display: grid;
  gap: 8px;
  margin-top: 10px;
}

.cycle-circuit-item {
  display: grid;
  align-items: center;
  gap: 6px;
  padding: 8px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
}

.cycle-circuit-item--simple {
  grid-template-columns: auto minmax(120px, 1fr) auto auto auto auto;
  column-gap: 6px;
}

.cycle-circuit-item input,
.cycle-circuit-item select {
  min-width: 0;
  min-height: 30px;
  border: 0;
  border-radius: 4px;
  padding: 4px 6px;
  background: #f7fbff;
  color: #111820;
  font-size: 0.72rem;
  font-weight: 800;
}

.cycle-circuit-item input[type="color"] {
  width: 42px;
  padding: 2px;
}

.cycle-circuit-item button {
  border: 0;
  border-radius: 4px;
  padding: 5px 7px;
  cursor: pointer;
  font-size: 0.7rem;
  font-weight: 800;
  background: #f7fbff;
  color: #111820;
}

.cycle-circuit-item button:hover {
  background: #d9f0ff;
}

.workflow-help-note {
  margin-top: 8px;
  padding: 8px 10px;
  border-radius: 6px;
  background: rgba(143, 211, 255, 0.12);
  color: #dbe9f1;
  font-size: 0.76rem;
  line-height: 1.35;
}

.global-color-legend {
  position: fixed;
  top: 24px;
  left: calc(23rem + 24px);
  z-index: 1002;
  width: min(260px, calc(100vw - 23rem - 48px));
  max-height: 42vh;
  overflow-y: auto;
  padding: 12px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.22);
  background: rgba(13, 22, 28, 0.88);
  color: #f7fbff;
  box-shadow: 0 16px 38px rgba(0, 0, 0, 0.24);
  backdrop-filter: blur(10px);
}

.global-color-legend__title {
  margin-bottom: 10px;
  color: #8fd3ff;
  font-size: 0.78rem;
  font-weight: 900;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.global-color-legend__list {
  display: grid;
  gap: 7px;
}

.global-color-legend__item {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 0;
  padding: 7px 8px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
}

.global-color-legend__text {
  display: grid;
  min-width: 0;
  color: #f7fbff;
  font-size: 0.76rem;
  font-weight: 900;
  line-height: 1.2;
}

.global-color-legend__text small {
  color: #b8c9d3;
  font-size: 0.66rem;
  font-weight: 700;
}

.global-color-legend__group {
  display: grid;
  gap: 6px;
}

.global-color-legend__cycle-title {
  margin-top: 6px;
  color: #8fd3ff;
  font-size: 0.72rem;
  font-weight: 900;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.global-color-legend__group:first-child .global-color-legend__cycle-title {
  margin-top: 0;
}

.ifc-panel-side-toggle {
  position: fixed;
  top: 76px;
  left: calc(23rem - 14px);
  z-index: 1003;
  display: inline-grid;
  width: 28px;
  height: 42px;
  place-items: center;
  border: 0;
  border-radius: 0 999px 999px 0;
  background: #f7fbff;
  color: #111820;
  cursor: pointer;
  font-size: 1.2rem;
  font-weight: 900;
  box-shadow: 0 8px 22px rgba(0, 0, 0, 0.28);
}

.ifc-panel-side-toggle:hover {
  background: #d9f0ff;
}

.ifc-panel-side-toggle--collapsed {
  left: 0;
  border-radius: 0 999px 999px 0;
}

.global-color-legend--ifc-collapsed {
  left: 48px;
}

.cycle-circuit-use-button--active {
  background: #8fd3ff !important;
  color: #07131a !important;
  box-shadow: 0 0 0 2px rgba(143, 211, 255, 0.35);
}

.cycle-circuit-lock-icon {
  font-size: 0.8rem;
  line-height: 1;
}

.cycle-circuit-item--locked {
  border: 1px solid rgba(143, 211, 255, 0.75);
  background: rgba(143, 211, 255, 0.14);
  box-shadow: 0 0 0 2px rgba(143, 211, 255, 0.14);
}

.cycle-circuit-item--locked input,
.cycle-circuit-item--locked select {
  background: rgba(143, 211, 255, 0.22);
  color: #f7fbff;
  border: 1px solid rgba(143, 211, 255, 0.55);
}

.cycle-circuit-lock-button--active {
  background: #8fd3ff !important;
  color: #07131a !important;
  box-shadow: 0 0 0 2px rgba(143, 211, 255, 0.35);
}

.cycle-circuit-name-wrapper {
  display: grid;
  grid-template-columns: auto minmax(0, 1fr);
  align-items: center;
  gap: 6px;
  min-width: 0;
}

.cycle-circuit-name-wrapper input {
  width: 100%;
}

.cycle-circuit-lock-icon {
  font-size: 0.8rem;
  line-height: 1;
}

@media (max-width: 820px) {
  .control-panels {
    top: auto;
    right: 12px;
    bottom: 12px;
    width: calc(100vw - 24px);
    max-height: calc(100vh - 24px);
  }

.global-color-legend {
  top: 12px;
  left: 12px;
  width: calc(100vw - 24px);
  max-height: 28vh;
}

  .corner-logo {
    display: none;
  }
}
</style>
