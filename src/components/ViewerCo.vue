<template>
  <div v-if="isLoading" class="loading-overlay">
    <div class="loading-box">
      <div class="spinner"></div>
      <p class="loading-text">Loading {{ loadingFileName }}</p>
      <p class="loading-text">{{ loadingProgress }}%</p>
    </div>
  </div>

  <div ref="containerRef" class="full-screen">

  <div
  v-if="isAutomaticValveAssociationPreviewOpen"
  class="automatic-valve-preview-backdrop"
>
  <div class="automatic-valve-preview-dialog">
    <div class="automatic-valve-preview-header">
      <div>
        <p class="flow-panel__eyebrow">
          Associação automática
        </p>

        <h2>
          Rever associações das válvulas
        </h2>
      </div>

      <button
        type="button"
        class="section-collapse-button"
        @click="
          closeAutomaticValveAssociationPreview
        "
      >
        ×
      </button>
    </div>

    <p class="connection-note">
      Confirma apenas as propostas corretas.
      Nenhuma associação foi ainda guardada.
    </p>

    <div
      class="flow-actions flow-actions--secondary"
    >
      <button
        type="button"
        @click="
          selectAllSafeValveAssociationPreviews
        "
      >
        Selecionar propostas seguras
      </button>

      <button
        type="button"
        @click="
          clearAutomaticValveAssociationPreviewSelection
        "
      >
        Desmarcar todas
      </button>
    </div>

    <p class="connection-note">
      Propostas selecionadas:
      {{
        getAcceptedAutomaticValvePreviewCount()
      }}
    </p>

    <div class="automatic-valve-preview-list">
      <article
        v-for="
          preview in
          automaticValveAssociationPreviews
        "
        :key="preview.valveKey"
        class="automatic-valve-preview-item"
        :class="{
          'automatic-valve-preview-item--safe':
            preview.status === 'safe',
          'automatic-valve-preview-item--warning':
            preview.status !== 'safe',
          'automatic-valve-preview-item--accepted':
            preview.accepted
        }"
      >
        <label
          class="automatic-valve-preview-selection"
        >
          <input
            type="checkbox"
            :checked="preview.accepted"
            :disabled="
              preview.status !== 'safe'
            "
            @change="
              toggleAutomaticValvePreviewAcceptance(
                preview
              )
            "
          />

          <strong>
            {{ preview.valveLabel }}
          </strong>
        </label>

        <dl class="automatic-valve-preview-details">
          <div>
            <dt>Tipo normal</dt>

            <dd>
              {{ preview.normalStateLabel }}
            </dd>
          </div>

          <div>
            <dt>Estado atual</dt>

            <dd>
              {{ preview.currentStateLabel }}
            </dd>
          </div>

          <div>
            <dt>Percurso proposto</dt>

            <dd>
              {{ preview.routeName }}
            </dd>
          </div>

          <div>
            <dt>Distância</dt>

            <dd>
              {{
                preview.distance === null
                  ? 'não calculada'
                  : preview.distance.toFixed(3) +
                    ' m'
              }}
            </dd>
          </div>

          <div>
            <dt>Tubos a jusante</dt>

            <dd>
              {{ preview.downstreamPipeCount }}
            </dd>
          </div>

          <div>
            <dt>Resultado</dt>

            <dd>
              {{
                getAutomaticValvePreviewStatusLabel(
                  preview.status
                )
              }}
            </dd>
          </div>
        </dl>

        <div
  class="flow-actions flow-actions--single"
>
  <button
    type="button"
    :disabled="
      !preview.routeId ||
      !preview.closestNode
    "
    @click="
      highlightAutomaticValveAssociationPreview(
        preview
      )
    "
  >
    Realçar proposta no modelo
  </button>
</div>
      </article>
    </div>

    <div
      class="flow-actions flow-actions--secondary"
    >
      <button
        type="button"
        @click="
          closeAutomaticValveAssociationPreview
        "
      >
        Cancelar
      </button>

      <button
  type="button"
  :disabled="
    getAcceptedAutomaticValvePreviewCount() === 0
  "
  @click="
    confirmAutomaticValveAssociationPreviews
  "
>
  Confirmar associações selecionadas
</button>
    </div>
  </div>
</div>

  <div
  v-if="selectedTubeRouteInfo"
  class="selected-tube-route-banner"
>
  <span class="selected-tube-route-banner__label">
    Percurso do elemento selecionado
  </span>

  <strong>
    {{ selectedTubeRouteInfo }}
  </strong>
</div>

  <div
  v-if="isRouteGroupDialogOpen"
  class="route-group-dialog-backdrop"
>
  <div class="route-group-dialog">
    <div class="route-group-dialog__header">
      <div>
        <p class="flow-panel__eyebrow">
  {{
    editingRouteGroupId
      ? 'Edição de grupo'
      : 'União de percursos'
  }}
</p>

       <h2>
  {{
    editingRouteGroupId
      ? 'Editar grupo de percursos'
      : 'Unir percursos'
  }}
</h2>

      </div>

      <button
        type="button"
        class="section-collapse-button"
        @click="closeRouteGroupingDialog"
      >
        ×
      </button>
    </div>

    <label class="route-group-dialog__field">
      <span>
  {{
    editingRouteGroupId
      ? 'Nome do grupo'
      : 'Nome do novo percurso'
  }}
</span>

      <input
        v-model="pendingRouteGroupName"
        type="text"
      />
    </label>

    <label class="route-group-dialog__field">
      <span>
  {{
    editingRouteGroupId
      ? 'Cor comum dos caminhos'
      : 'Cor do novo percurso'
  }}
</span>

      <div class="route-group-color-control">
        <input
          v-model="pendingRouteGroupColor"
          type="color"
        />

        <span
          class="route-group-color-preview"
          :style="{
            backgroundColor:
              pendingRouteGroupColor
          }"
        ></span>

        <strong>
  {{
    editingRouteGroupId
      ? 'Esta será a cor dos caminhos.'
      : 'Esta será a cor do novo percurso.'
  }}
</strong>
      </div>
    </label>

    <p class="connection-note">
  <template v-if="editingRouteGroupId">
    A nova cor será aplicada a todos os caminhos
    deste grupo. Ao retirar um caminho do grupo,
    a respetiva cor original será reposta.
  </template>

  <template v-else>
  Serão unidos
  {{ selectedRouteIdsForGrouping.size }}
  percursos num único percurso. Os tubos
  repetidos serão incluídos apenas uma vez.
  Poderás desfazer a união posteriormente.
</template>
</p>

    <div class="flow-actions flow-actions--secondary">
      <button
        type="button"
        @click="closeRouteGroupingDialog"
      >
        Cancelar
      </button>

      <button
  type="button"
  class="automatic-review-button"
  @click="saveRouteGroupChanges"
>
  {{
  editingRouteGroupId
    ? 'Guardar alterações'
    : 'Unir percursos'
}}
</button>
    </div>
  </div>
</div>

<div
  v-if="isRouteColorDialogOpen"
  class="route-group-dialog-backdrop"
>
  <div class="route-group-dialog">
    <div class="route-group-dialog__header">
      <div>
        <p class="flow-panel__eyebrow">
          Cor individual
        </p>

        <h2>Alterar cor do percurso</h2>
      </div>

      <button
        type="button"
        class="section-collapse-button"
        @click="closeIndividualRouteColorDialog"
      >
        ×
      </button>
    </div>

    <label class="route-group-dialog__field">
      <span>Nova cor do caminho</span>

      <div class="route-group-color-control">
        <input
          v-model="pendingIndividualRouteColor"
          type="color"
        />

        <span
          class="route-group-color-preview"
          :style="{
            backgroundColor:
              pendingIndividualRouteColor
          }"
        ></span>

        <strong>
          Esta cor será aplicada apenas
          a este caminho.
        </strong>
      </div>
    </label>

    <p class="connection-note">
      Podes utilizar “Repor cor” para voltar
      à cor original do circuito.
    </p>

    <div class="flow-actions flow-actions--secondary">
      <button
        type="button"
        @click="closeIndividualRouteColorDialog"
      >
        Cancelar
      </button>

      <button
        type="button"
        class="automatic-review-button"
        @click="saveIndividualRouteColor"
      >
        Guardar cor
      </button>
    </div>
  </div>
</div>

<div
  v-if="isSavedRouteDirectionPanelOpen"
  class="route-direction-dialog-backdrop"
>
  <div class="route-direction-dialog">
    <div class="route-direction-dialog__header">
      <div>
        <p class="flow-panel__eyebrow">
          Orientação do fluxo
        </p>

        <h2>Definir sentido do percurso</h2>
      </div>

      <button
        type="button"
        class="section-collapse-button"
        @click="closeSavedRouteDirectionPanel"
      >
        ×
      </button>
    </div>

    <div class="route-direction-dialog__route">
      <span>Percurso selecionado</span>

      <strong>
        {{
          getSelectedSavedRouteForDirection()?.name
        }}
      </strong>
    </div>

    <p class="route-direction-dialog__help">
      Seleciona um tubo no modelo e define-o como
      início. Depois seleciona outro tubo e define-o
      como fim.
    </p>

    <p
  v-if="
    savedRouteDirectionWarning
  "
  class="
    route-direction-dialog__warning
  "
>
  {{ savedRouteDirectionWarning }}
</p>

    <div class="route-direction-dialog__definition-grid">
      <div class="route-direction-dialog__definition">
  <span class="route-direction-dialog__step">
    1
  </span>

  <strong>Inícios do fluxo</strong>

  <div
    v-if="savedRouteDirectionStarts.length"
    class="route-direction-dialog__node-list"
  >
    <div
      v-for="node in savedRouteDirectionStarts"
      :key="
        `direction-start-${node.modelId}-${node.localId}`
      "
      class="route-direction-dialog__node"
    >
      <span>
        Tubo #{{ node.localId }}
      </span>

      <button
        type="button"
        title="Remover início"
        @click="
          removeSavedRouteDirectionStart(
            node
          )
        "
      >
        ×
      </button>
    </div>
  </div>

  <span
    v-else
    class="route-direction-dialog__value"
  >
    Nenhum início definido
  </span>

  <button
    type="button"
    @click="setSavedRouteDirectionStart"
  >
    Adicionar início selecionado
  </button>
</div>

      <div class="route-direction-dialog__definition">
  <span class="route-direction-dialog__step">
    2
  </span>

  <strong>Fins do fluxo</strong>

  <div
    v-if="savedRouteDirectionEnds.length"
    class="route-direction-dialog__node-list"
  >
    <div
      v-for="node in savedRouteDirectionEnds"
      :key="
        `direction-end-${node.modelId}-${node.localId}`
      "
      class="route-direction-dialog__node"
    >
      <span>
        Tubo #{{ node.localId }}
      </span>

      <button
        type="button"
        title="Remover fim"
        @click="
          removeSavedRouteDirectionEnd(
            node
          )
        "
      >
        ×
      </button>
    </div>
  </div>

  <span
    v-else
    class="route-direction-dialog__value"
  >
    Nenhum fim definido
  </span>

  <button
    type="button"
    @click="setSavedRouteDirectionEnd"
  >
    Adicionar fim selecionado
  </button>
</div>
    </div>

    <div class="route-direction-dialog__actions">
      <button
        type="button"
        @click="closeSavedRouteDirectionPanel"
      >
        Cancelar
      </button>

      <button
  type="button"
  class="automatic-review-button"
  :disabled="
    savedRouteDirectionStarts.length === 0 ||
    savedRouteDirectionEnds.length === 0 ||
    getSelectedSavedRouteForDirection()?.locked
  "
  @click="applyAndSaveSavedRouteDirection"
>
  Aplicar e guardar sentido
</button>
    </div>
  </div>
</div>

  <nav
  class="application-tabs"
  :class="{
    'application-tabs--ifc-collapsed':
      isIfcPanelCollapsed
  }"
>
  <button
    type="button"
    :class="[
      'application-tab',
      activeApplicationTab === 'automatic'
        ? 'application-tab--active'
        : ''
    ]"
    @click="activeApplicationTab = 'automatic'"
  >
    Análise Automática
  </button>

  <button
    type="button"
    :class="[
      'application-tab',
      activeApplicationTab === 'manual'
        ? 'application-tab--active'
        : ''
    ]"
    @click="activeApplicationTab = 'manual'"
  >
    Configuração manual
  </button>

  <button
    type="button"
    :class="[
      'application-tab',
      activeApplicationTab === 'simulation'
        ? 'application-tab--active'
        : ''
    ]"
    @click="activeApplicationTab = 'simulation'"
  >
    Simulação
  </button>
</nav>

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
  v-if="hasLoadedModel && cycleCircuitDefinitions.length"
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
  v-if="activeApplicationTab === 'automatic'"
  class="flow-panel automatic-analysis-panel"
  aria-label="Análise automática"
>
  <div class="flow-panel__header">
    <div>
      <p class="flow-panel__eyebrow">Configuração automática</p>
      <h2>Análise do IFC</h2>
    </div>
  </div>

  <div class="flow-panel__content">
    <p
  v-if="!hasLoadedModel"
  class="workflow-help-note"
>
  Carrega um ficheiro IFC no painel da esquerda para iniciar a análise automática.
</p>

<p
  v-else-if="
  activeIfcStorageId &&
  savedRoutes.length
"
  class="automatic-analysis-status automatic-analysis-status--ready"
>
  ✓ {{ savedRoutes.length }} caminho(s) guardado(s)
  aplicado(s) automaticamente. Não é necessário repetir
  o scan se estás a usar o mesmo IFC.
</p>

<p
  v-else
  class="automatic-analysis-status automatic-analysis-status--ready"
>
  IFC carregado. Ainda não existem caminhos guardados
  para aplicar.
</p>

<div
  v-if="hasLoadedModel"
  class="flow-actions flow-actions--single"
>
  <button
    type="button"
    :disabled="isSystemScanRunning"
    @click="scanIfcSystems"
  >
    {{
      isSystemScanRunning
        ? 'A analisar sistemas...'
        : savedRoutes.length
          ? 'Voltar a fazer scan do IFC'
          : 'Analisar System Types e System Names'
    }}
  </button>
</div>

<div
  v-if="hasSystemScanResults"
  class="automatic-analysis-results"
>
  <div class="automatic-system-result">
    <div class="automatic-system-result__header">
      <span>
        System Types diferentes
      </span>

      <strong>
        {{ systemScanResults.systemTypes.length }}
      </strong>

      <button
        type="button"
        class="section-collapse-button"
        @click="
          isSystemTypesListOpen =
            !isSystemTypesListOpen
        "
      >
        {{ isSystemTypesListOpen ? '−' : '+' }}
      </button>
    </div>

    <div
      v-if="isSystemTypesListOpen"
      class="automatic-system-result__list"
    >
      <span
        v-for="systemType in systemScanResults.systemTypes"
        :key="`system-type-${systemType}`"
      >
        {{ systemType }}
      </span>

      <span
        v-if="!systemScanResults.systemTypes.length"
        class="automatic-system-result__empty"
      >
        Nenhum System Type encontrado.
      </span>
    </div>
  </div>

  <div class="automatic-system-result">
    <div class="automatic-system-result__header">
      <span>
        System Names diferentes
      </span>

      <strong>
        {{ systemScanResults.systemNames.length }}
      </strong>

      <button
        type="button"
        class="section-collapse-button"
        @click="
          isSystemNamesListOpen =
            !isSystemNamesListOpen
        "
      >
        {{ isSystemNamesListOpen ? '−' : '+' }}
      </button>
    </div>

    <div
      v-if="isSystemNamesListOpen"
      class="automatic-system-result__list"
    >
      <span
        v-for="systemName in systemScanResults.systemNames"
        :key="`system-name-${systemName}`"
      >
        {{ systemName }}
      </span>

      <span
        v-if="!systemScanResults.systemNames.length"
        class="automatic-system-result__empty"
      >
        Nenhum System Name encontrado.
      </span>
    </div>
  </div>
</div>

<div
  v-if="
    hasSystemScanResults &&
    systemScanResults.systemNames.length
  "
  class="flow-actions flow-actions--single"
>
  <button
    type="button"
    @click="createAutomaticCircuitsFromSystemNames"
  >
    Criar caminhos a partir dos System Names
  </button>
</div>

<div
  v-if="automaticOrderedCircuitNodes.size"
  class="flow-actions flow-actions--single"
>
  <button
    type="button"
    @click="saveAutomaticRoutes"
  >
    Guardar caminhos automáticos
  </button>
</div>

   <div
  v-if="hasLoadedModel"
  class="flow-actions flow-actions--single"
>
  <button
    type="button"
    :disabled="isAutomaticAnalysisRunning"
    @click="startAutomaticAnalysis"
  >
    {{
      isAutomaticAnalysisRunning
        ? 'A analisar IFC...'
        : 'Iniciar análise automática'
    }}
  </button>
</div>

<div
  v-if="hasAutomaticAnalysisResults"
  class="automatic-analysis-results"
>
  <div class="automatic-analysis-result">
    <span>Tubos e acessórios</span>
    <strong>{{ automaticAnalysisResults.pipes }}</strong>
  </div>

  <div class="automatic-analysis-result">
    <span>Válvulas e controladores</span>
    <strong>{{ automaticAnalysisResults.valves }}</strong>
  </div>

  <div class="automatic-analysis-result">
    <span>Equipamentos</span>
    <strong>{{ automaticAnalysisResults.equipment }}</strong>
  </div>

  <div
    class="automatic-analysis-result automatic-analysis-result--total"
  >
    <span>Total identificado</span>
    <strong>{{ automaticAnalysisResults.total }}</strong>
  </div>
</div>
<div
  v-if="hasAutomaticAnalysisResults"
  class="flow-actions flow-actions--single"
>
  <button
    type="button"
    class="flow-button--danger"
    @click="clearAutomaticAnalysisResults"
  >
    Limpar resultados automáticos
  </button>
</div>
  </div>
</section>
      <section
  v-if="false"
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

<div class="flow-actions flow-actions--single">
  <button type="button" @click="extractSelectedIfcInformation">
    Extrair dados IFC do selecionado
  </button>
</div>

<div
  v-if="selectedIfcDetailsText"
  class="flow-section-title flow-section-title--button"
>
  <span>Dados IFC extraídos</span>
  <button
    type="button"
    class="section-collapse-button"
    @click="isIfcDetailsPanelOpen = !isIfcDetailsPanelOpen"
  >
    {{ isIfcDetailsPanelOpen ? '−' : '+' }}
  </button>
</div>

<pre
  v-if="selectedIfcDetailsText && isIfcDetailsPanelOpen"
  class="ifc-details-output"
>{{ selectedIfcDetailsText }}</pre>

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
  v-if="
    activeApplicationTab === 'manual'
  "
  :class="[
    'flow-panel',
    {
      'flow-panel--minimized':
        isIfcInformationPanelMinimized
    }
  ]"
  aria-label="Dados IFC do elemento selecionado"
>
  <div class="flow-panel__header">
    <div>
      <p class="flow-panel__eyebrow">
        Elemento selecionado
      </p>

      <h2>
        Dados IFC
      </h2>
    </div>

    <button
      type="button"
      class="flow-panel__toggle"
      @click="
        toggleIfcInformationPanelMinimized
      "
    >
      {{
        isIfcInformationPanelMinimized
          ? '+'
          : '−'
      }}
    </button>
  </div>

  <div
    v-if="
      !isIfcInformationPanelMinimized
    "
    class="flow-panel__content"
  >
    <p class="selection-count">
      Selecionados:
      {{ selectedCount }}
    </p>

    <div class="selected-mep-info">
      {{ selectedMepElementInfo }}
    </div>

    <p class="connection-note">
      Seleciona um elemento no modelo para
      consultar os respetivos dados IFC.
    </p>

    <div
      class="
        flow-actions
        flow-actions--single
      "
    >
      <button
        type="button"
        :disabled="
          selectedCount === 0
        "
        @click="
          extractSelectedIfcInformation
        "
      >
        Extrair dados IFC do selecionado
      </button>
    </div>

    <div
      v-if="selectedIfcDetailsText"
      class="
        flow-section-title
        flow-section-title--button
      "
    >
      <span>
        Dados IFC extraídos
      </span>

      <button
        type="button"
        class="section-collapse-button"
        @click="
          isIfcDetailsPanelOpen =
            !isIfcDetailsPanelOpen
        "
      >
        {{
          isIfcDetailsPanelOpen
            ? '−'
            : '+'
        }}
      </button>
    </div>

    <pre
      v-if="
        selectedIfcDetailsText &&
        isIfcDetailsPanelOpen
      "
      class="ifc-details-output"
    >{{ selectedIfcDetailsText }}</pre>
  </div>
</section>

<section
  v-if="activeApplicationTab === 'manual'"
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

<div
  class="
    flow-section-title
    flow-section-title--button
  "
>
  <span>
    Configurar ciclos e circuitos
  </span>

  <button
    type="button"
    class="section-collapse-button"
    @click="
      isCycleConfigurationSectionOpen =
        !isCycleConfigurationSectionOpen
    "
  >
    {{
      isCycleConfigurationSectionOpen
        ? '−'
        : '+'
    }}
  </button>
</div>

<div
  v-if="
    isCycleConfigurationSectionOpen
  "
>

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
  1. Escolher ciclo e circuito
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
  Este ciclo ainda não tem circuitos. Cria um circuito em baixo para começar.
</p>

<div class="flow-section-title flow-section-title--button">
  <span>Definições dos circuitos deste ciclo</span>
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
    <span>Tipo de circuito</span>
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
    <span>Nome do circuito</span>
    <input
  v-model="pendingCycleCircuitName"
  type="text"
  placeholder="Ex: Nome do circuito"
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
  Criar e selecionar circuito
</button>

    <button type="button" @click="saveCycleCircuitDefinitionChanges">
      Guardar alterações
    </button>
  </div>

  <p
  v-if="!getActiveCycleCircuitDefinitions().length"
  class="connection-note workflow-help-note"
>
  Este ciclo ainda não tem circuitos. Cria um circuito para começar.
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
  class="saved-route-action--reset-color"
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

</div>

<div
  class="
    flow-section-title
    flow-section-title--button
  "
>
  <span>
    Ações sobre tubos selecionados
  </span>

  <button
    type="button"
    class="section-collapse-button"
    @click="
      isSelectedPipeActionsSectionOpen =
        !isSelectedPipeActionsSectionOpen
    "
  >
    {{
      isSelectedPipeActionsSectionOpen
        ? '−'
        : '+'
    }}
  </button>
</div>

<div
  v-if="
    isSelectedPipeActionsSectionOpen
  "
>
  <p class="connection-note">
  Tubos/elementos selecionados:
  <strong>
    {{ selectedCount }}
  </strong>
</p>

<p class="connection-note">
  Estas ações atuam apenas nos tubos
  atualmente selecionados no modelo.
</p>

    <div class="flow-actions flow-actions--secondary">
      <button
  type="button"
  :disabled="
    selectedCount === 0
  "
  @click="
    clearCurrentModelSelection
  "
>
  Limpar seleção atual
</button>

      <button type="button" @click="clearManualAssignments">
      Limpar todas as marcações
      </button>

      <button
  type="button"
  :disabled="
    selectedCount === 0
  "
  @click="
    reverseSelectedPipesDirection
  "
>
  Sincronizar sentido
</button>
    </div>

<div class="selected-pipe-arrow-actions">
  <button
    type="button"
    :disabled="
      selectedCount === 0
    "
    @click="
      hideSelectedFlowArrows
    "
  >
    Ocultar setas
  </button>

  <button
    type="button"
    :disabled="
      selectedCount === 0
    "
    @click="
      showSelectedFlowArrows
    "
  >
    Mostrar setas
  </button>
</div>
    </div>

    <div
  class="
    flow-section-title
    flow-section-title--button
  "
>
  <span>
    Editar caminho existente
  </span>

  <button
    type="button"
    class="section-collapse-button"
    @click="
      isEditRouteSectionOpen =
        !isEditRouteSectionOpen
    "
  >
    {{
      isEditRouteSectionOpen
        ? '−'
        : '+'
    }}
  </button>
</div>

<div v-if="isEditRouteSectionOpen">

<label class="flow-cycle-config">
  <span>
    Caminho a editar
  </span>

  <select
  v-model="
    selectedSavedRouteIdForEditing
  "
  @change="
    highlightSavedRouteForEditing
  "
>
    <option value="">
      Selecionar caminho...
    </option>

    <option
  v-for="
    route in
    getAllSavedRoutesOrdered()
  "
      :key="
        'edit-saved-route-' +
        route.id
      "
      :value="route.id"
      :disabled="route.locked"
    >
      {{
        route.name +
        (
          route.locked
            ? ' · protegido'
            : ''
        )
      }}
    </option>
  </select>
</label>

<div class="route-edit-help">
  <p class="route-edit-help__title">
    Como editar o caminho
  </p>

  <ol class="route-edit-help__steps">
    <li>
      Seleciona no modelo os tubos que queres
      adicionar ou retirar.
    </li>

    <li>
      Clica na operação correspondente.
    </li>
  </ol>

  <p class="route-edit-help__warning">
    Ao adicionar tubos, será necessário definir
    novamente o sentido do caminho.
  </p>
</div>
<div
  class="flow-actions flow-actions--single"
>
  <button
    type="button"
    :disabled="
      !selectedSavedRouteIdForEditing ||
      selectedCount === 0
    "
    @click="
      addSelectedPipesToEditedRoute
    "
  >
    Adicionar tubos selecionados ao caminho
  </button>

  <button
    type="button"
    :disabled="
      !selectedSavedRouteIdForEditing ||
      selectedCount === 0
    "
    @click="
      removeSelectedPipesFromEditedRoute
    "
  >
    Retirar tubos selecionados do caminho
  </button>
</div>
</div>

<div
  class="
    flow-section-title
    flow-section-title--button
  "
>
  <span>
    Criar novo caminho
  </span>

  <button
    type="button"
    class="section-collapse-button"
    @click="
      isCreateRouteSectionOpen =
        !isCreateRouteSectionOpen
    "
  >
    {{
      isCreateRouteSectionOpen
        ? '−'
        : '+'
    }}
  </button>
</div>

<div v-if="isCreateRouteSectionOpen">
  <div class="flow-section-title">
    1. Ciclo do caminho
  </div>

  <label class="flow-cycle-config">
    <span>
      Ciclo
    </span>

    <select
      v-model="
        newRouteCycleSelection
      "
    >
      <option value="">
        Sem ciclo / Por atribuir
      </option>

      <option
        v-for="
          cycleNumber in
          waterCycleCount
        "
        :key="
          'new-route-cycle-' +
          cycleNumber
        "
        :value="
          String(cycleNumber)
        "
      >
        {{
          getCycleDisplayName(
            cycleNumber
          )
        }}
      </option>
    </select>
  </label>

  <label class="flow-cycle-config">
  <span>
    Tipo do novo circuito
  </span>

  <select
    v-model="
      newRouteCircuitKind
    "
    @change="
      handleNewRouteCircuitKindChange
    "
  >
    <option value="hotSupply">
      Ida quente
    </option>

    <option value="coldSupply">
      Ida fria
    </option>

    <option value="hotReturn">
      Retorno quente
    </option>

    <option value="coldReturn">
      Retorno frio
    </option>

    <option value="extra">
      Extra
    </option>
  </select>
</label>

<label class="flow-cycle-config">
  <span>
    Nome do novo circuito
  </span>

  <input
    v-model="
      newRouteCircuitName
    "
    type="text"
    :placeholder="
      getDefaultNewRouteCircuitName(
        newRouteCircuitKind
      )
    "
  />
</label>

<label class="flow-cycle-config">
  <span>
    Cor do novo circuito
  </span>

  <input
    v-model="
      newRouteCircuitColor
    "
    type="color"
  />
</label>

  <p
    v-if="
      !newRouteCycleSelection
    "
    class="connection-note"
  >
    O caminho será criado fora dos ciclos.
    Podes atribuí-lo posteriormente.
  </p>

    class="
      connection-note
      workflow-help-note
    "
  >

  <div class="flow-section-title">
    2. Método de criação
  </div>

  <div class="route-creation-method-switch">
    <button
      type="button"
      :class="{
        'route-creation-method-switch__option--active':
          routeCreationMethod ===
          'automatic'
      }"
      @click="
        selectRouteCreationMethod(
          'automatic'
        )
      "
    >
      Por início e fim
    </button>

    <button
      type="button"
      :class="{
        'route-creation-method-switch__option--active':
          routeCreationMethod ===
          'manual'
      }"
      @click="
        selectRouteCreationMethod(
          'manual'
        )
      "
    >
      Tubo a tubo
    </button>
  </div>

  <p class="connection-note">
    {{
      routeCreationMethod ===
        'automatic'
        ? 'Define o início e o fim. O programa encontra automaticamente os tubos entre ambos.'
        : 'Seleciona manualmente cada tubo pela ordem do caminho.'
    }}
  </p>

  <div
    v-if="
      routeCreationMethod ===
      'automatic'
    "
  >
    <div
      class="route-creation-status"
    >
      <p>
        <strong>
          Início:
        </strong>

        {{ routeStartLabel }}
      </p>

      <p>
        <strong>
          Fim:
        </strong>

        {{ routeEndLabel }}
      </p>
    </div>

<div class="route-creation-three-actions">
  <button
    type="button"
    @click="
      setRouteStart
    "
  >
    Definir início
  </button>

  <button
    type="button"
    @click="
      setRouteEnd
    "
  >
    Definir fim
  </button>

  <button
    type="button"
    :disabled="
  !routeStart ||
  !routeEnd
"
    @click="
      createAutoRouteForSelectedCycleCircuit
    "
  >
    Criar caminho
  </button>
</div>
  </div>

  <div
    v-if="
      routeCreationMethod ===
      'manual'
    "
  >
    <div
      class="
        flow-actions
        flow-actions--single
      "
    >
      <button
        type="button"
        :class="[
          'manual-route-mode-button',
          isManualRouteRecording
            ? 'manual-route-mode-button--active'
            : ''
        ]"
        @click="
          toggleManualRouteRecording
        "
      >
        {{
          isManualRouteRecording
            ? 'Terminar seleção de tubos'
            : 'Iniciar seleção de tubos'
        }}
      </button>
    </div>

    <p
      v-if="
        isManualRouteRecording
      "
      class="manual-route-status"
    >
      Seleção ativa: clica nos tubos
      pela ordem do caminho.
    </p>

    <div
      class="route-creation-status"
    >
      <p>
        <strong>
          Tubos adicionados:
        </strong>

        {{ manualRouteNodes.length }}
      </p>

      <p>
        <strong>
          Início proposto:
        </strong>

        {{
          manualRouteNodes.length
            ? formatNodeLabel(
                manualRouteNodes[0]
              )
            : 'nenhum'
        }}
      </p>

      <p>
        <strong>
          Fim proposto:
        </strong>

        {{
          manualRouteNodes.length
            ? formatNodeLabel(
                manualRouteNodes[
                  manualRouteNodes.length -
                  1
                ]
              )
            : 'nenhum'
        }}
      </p>
    </div>

  <div class="route-creation-three-actions">
  <button
    type="button"
    :disabled="
      manualRouteNodes.length ===
      0
    "
    @click.stop.prevent="
      removeLastManualRouteNode
    "
  >
    Remover último tubo
  </button>

  <button
  type="button"
  :disabled="
    manualRouteNodes.length ===
    0
  "
  @click.stop.prevent="
    startManualRouteRecording
  "
>
  Limpar seleção
</button>

  <button
    type="button"
    :disabled="
  manualRouteNodes.length < 2
"
    @click="
      createManualRouteForSelectedCycleCircuit
    "
  >
    Criar caminho
  </button>
</div>
  </div>

  <div
    v-if="
      currentRouteConnections.length >
      0
    "
    class="flow-section-title"
  >
    3. Guardar ou descartar
  </div>

  <div
    v-if="
      currentRouteConnections.length >
      0
    "
    class="
      flow-actions
      flow-actions--single
    "
  >
    <button
      type="button"
      class="flow-button--primary"
      @click="
        saveCurrentRoute
      "
    >
      Guardar caminho
    </button>

    <button
      type="button"
      class="flow-button--danger"
      @click="
        discardCurrentRoute
      "
    >
      Descartar caminho atual
    </button>
  </div>

  <p
    v-if="discardRouteMessage"
    class="discard-route-message"
  >
    {{ discardRouteMessage }}
  </p>
</div>

 <div v-if="hasLoadedModel && savedRoutes.length" class="saved-routes">
  <div class="saved-routes-header">
  <span>
    Percursos guardados: {{ savedRoutes.length }}
  </span>

  <button
    type="button"
    class="section-collapse-button"
    @click="toggleSavedRoutesPanel"
  >
    {{ isSavedRoutesPanelOpen ? '−' : '+' }}
  </button>
</div>

  <div
  v-if="isSavedRoutesPanelOpen"
  class="saved-route-group-actions"
>

<label class="saved-route-search">
  <span>
    Procurar percurso
  </span>

  <div class="saved-route-search__control">
    <input
      v-model="savedRouteSearchText"
      type="search"
      placeholder="Escreve o nome do percurso..."
    />
  </div>
</label>

<p
  v-if="savedRouteSearchText"
  class="connection-note"
>
  {{ getFilteredSavedRoutes().length }}
  de
  {{ savedRoutes.length }}
  percurso(s) encontrado(s)
</p>

  <span>
    Selecionados:
    {{ selectedRouteIdsForGrouping.size }}
  </span>

  <label class="saved-route-cycle-selector">
  <span>Ciclo de destino</span>

  <select
    v-model.number="
      targetCycleNumberForSavedRoutes
    "
  >
    <option
      v-for="cycleNumber in waterCycleCount"
      :key="`saved-route-target-cycle-${cycleNumber}`"
      :value="cycleNumber"
    >
      {{ getCycleDisplayName(cycleNumber) }}
    </option>
  </select>
</label>

<button
  type="button"
  :disabled="
    selectedRouteIdsForGrouping.size < 1
  "
  @click="moveSelectedSavedRoutesToCycle"
>
  Mover selecionados para o ciclo
</button>

<button
  type="button"
  :disabled="
    selectedRouteIdsForGrouping.size !== 2
  "
  @click="
    compareSelectedSavedRoutes
  "
>
  Comparar percursos
</button>

  <button
  type="button"
  :disabled="
    selectedRouteIdsForGrouping.size < 2
  "
  @click="openRouteGroupingDialog"
>
  Unir percursos selecionados
</button>

  <button
  type="button"
  :disabled="
    selectedRouteIdsForGrouping.size < 1
  "
  @click="
    clearRouteGroupingSelection
  "
>
  Limpar seleção
</button>

<button
  type="button"
  class="flow-button--danger"
  :disabled="
    selectedRouteIdsForGrouping.size < 1
  "
  @click="
    deleteSelectedSavedRoutes
  "
>
  Apagar percursos selecionados
</button>
</div>

  <div v-if="isSavedRoutesPanelOpen" class="saved-routes-list">

  <p
  v-if="
    savedRouteSearchText &&
    getFilteredSavedRoutes().length === 0
  "
  class="saved-route-search__empty"
>
  Não foi encontrado nenhum percurso com esse nome.
</p>

  <div
    v-for="route in getFilteredSavedRoutes()"
    :key="route.id"
    class="saved-route-item"
      :class="{
  'saved-route-item--locked': route.locked,  
  'saved-route-item--highlighted': highlightedSavedRouteId === route.id,
  'saved-route-item--blocked': isSavedRouteBlocked(route)
}"
    >

    <div class="saved-route-top-line">
  <label class="saved-route-group-checkbox">
    <input
      type="checkbox"
      :checked="
        isRouteSelectedForGrouping(
          route.id
        )
      "
      :disabled="route.locked"
      @change="
        toggleRouteSelectionForGrouping(
          route.id
        )
      "
    />

    <span>
  {{
    route.locked
      ? 'Seleção bloqueada'
      : 'Selecionar'
  }}
</span>
  </label>

  <span
    v-if="
      isRouteSelectedForSimulation(
        route.id
      )
    "
    class="saved-route-simulation-status"
    title="Percurso incluído na simulação"
    aria-label="Percurso incluído na simulação"
  ></span>
</div>

      <div class="saved-route-select saved-route-select--details">
  <span class="saved-route-text">
  <span
  class="saved-route-group-color"
  :style="{
    backgroundColor:
      getSavedRouteGroupColor(route)
  }"
></span>
          <strong>
  <span
    v-if="route.locked"
    class="route-lock-icon"
  >
    🔒
  </span>

  <span
    v-if="isSavedRouteBlocked(route)"
    class="route-partial-block-icon"
    :title="
      'Este percurso contém tubos bloqueados por válvulas. ' +
      getValveLabelForBlockedRoute(route)
    "
  >
    ◉
  </span>

  {{ route.name }}
</strong>

          <small>
            {{ getRouteCircuitDisplayLabel(route) }} ·
{{ getRoutePipeCount(route) }} tubo(s) ·
{{ getRouteVisibilityLabel(route) }} ·
{{ getRouteProtectionLabel(route) }}

<span v-if="isSavedRouteBlocked(route)">
  · Bloqueio parcial
</span>

<span
  v-if="
    route.needsDirectionRedefinition
  "
>
  · Sentido por definir
</span>

          </small>
        </span>
      </div>

      <div
  class="saved-route-actions"
  :class="{
    'saved-route-actions--locked':
      route.locked
  }"
>

        <button 
  type="button"
  class="saved-route-action--highlight"
  @click="toggleSavedRouteHighlight(route)"
>
  {{ highlightedSavedRouteId === route.id ? 'Limpar realce' : 'Realçar' }}
</button>

        <button
          v-if="!route.hidden"
          type="button"
          class="saved-route-action--visibility"
          @click="setSavedRouteVisibility(route.id, false)"
        >
          Ocultar
        </button>

        <button
  v-else
          type="button"
          class="saved-route-action--visibility"
          @click="setSavedRouteVisibility(route.id, true)"
        >
          Mostrar
        </button>

        <button
  v-if="!route.locked"
  type="button"
  class="saved-route-action--arrows"
  :disabled="selectedCount === 0"
  @click="
    toggleSelectedFlowArrows
  "
>
  {{
    areSelectedFlowArrowsHidden()
      ? 'Mostrar setas selecionadas'
      : 'Ocultar setas selecionadas'
  }}
</button>

        <button
  v-if="!route.locked"
  type="button"
  class="saved-route-action--direction"
  @click="openSavedRouteDirectionPanel(route)"
>
  Definir sentido
</button>

<button
  v-if="!route.locked"
  type="button"
  class="saved-route-action--sync"
  @click="
    syncSelectedPipesForSavedRoute(
      route
    )
  "
>
  Sincronizar sentido
</button>

        <button
          v-if="!route.locked"
          type="button"
          class="saved-route-action--rename"
          @click="renameSavedRoute(route.id)"
        >
          Renomear
        </button>

        <button
  v-if="
  route.groupId &&
  !route.locked
"
  type="button"
  class="saved-route-action--color"
  @click="openRouteGroupColorDialog(route)"
>
  Alterar cor do grupo
</button>

<button
  v-if="
    !route.groupId &&
    !route.locked
  "
  type="button"
  class="saved-route-action--color"
  @click="
    openIndividualRouteColorDialog(
      route
    )
  "
>
  Alterar cor
</button>

<button
  v-if="
    !route.groupId &&
    !route.locked
  "
  type="button"
  class="saved-route-action--reset-color"
  :disabled="
    !route.customColor
  "
  :title="
    route.customColor
      ? 'Repor a cor original do caminho'
      : 'A cor deste caminho não foi alterada'
  "
  @click="
    resetIndividualRouteColor(
      route
    )
  "
>
  Repor cor
</button>

        <button
  type="button"
  class="saved-route-action--protect"
  @click="
    toggleSavedRouteProtection(
      route.id
    )
  "
>
          {{ route.locked ? 'Desproteger' : 'Proteger' }}
        </button>

        <button
  type="button"
  :class="[
  'saved-route-action--simulation',
  {
    'saved-route-simulation-button--active':
      isRouteSelectedForSimulation(
        route.id
      )
  }
]"

  @click="
    toggleRouteForSimulation(
      route.id
    )
  "
>
  {{
    isRouteSelectedForSimulation(
      route.id
    )
      ? 'Na simulação'
      : 'Simular'
  }}
</button>

<button
  v-if="!route.locked"
  type="button"
  class="saved-route-action--undo-merge"
  :disabled="
    !route.mergeBackup
  "
  :title="
    route.mergeBackup
      ? 'Recuperar os caminhos anteriores à união'
      : 'Este caminho não foi criado através de uma união'
  "
  @click="
    undoSavedRouteMerge(
      route.id
    )
  "
>
  Desfazer união
</button>

<button
  v-if="!route.locked"
  type="button"
  class="
    saved-route-action--delete
    flow-button--danger
  "
  @click="
    deleteSavedRoute(
      route.id
    )
  "
>
  Apagar
</button>
      </div>
    </div>
  </div>
</div>

    <div class="flow-section-title flow-section-title--button">
  <span>Estatísticas dos caminhos</span>

  <button
    type="button"
    class="section-collapse-button"
    @click="isPathStatsPanelOpen = !isPathStatsPanelOpen"
  >
    {{ isPathStatsPanelOpen ? '−' : '+' }}
  </button>
</div>

    <dl
  v-if="isPathStatsPanelOpen"
  class="flow-stats"
>
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

<p
  v-if="isPathStatsPanelOpen"
  class="flow-note"
>
  {{ flowMessage }}
</p>
  </div>
</section>

    <section
  v-if="activeApplicationTab === 'manual'"
  :class="[
    'flow-panel',
    isSimulationControlPanelMinimized ? 'flow-panel--minimized' : ''
  ]"
  aria-label="Gestão e controlo de válvulas"
>  
        <div class="flow-panel__header">
          <div>
            <p class="flow-panel__eyebrow">
  Elementos hidráulicos
</p>

<h2>
  Válvulas
</h2>
          </div>

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
  <span>
  Selecionar e controlar válvulas
</span>

  <button
    type="button"
    class="section-collapse-button"
    @click="toggleValveDesignationPanel"
  >
    {{ isValveDesignationPanelOpen ? '−' : '+' }}
  </button>
</div>

<div v-if="isValveDesignationPanelOpen">
<p class="connection-note workflow-help-note">
  Escolhe uma válvula para a localizar, consultar
  o estado atual, abrir, fechar ou editar.
  A indicação NA/NF é apenas informativa.
</p>

<div class="valve-designation-dropdown">
  <span class="valve-designation-dropdown__label">
  Válvula
</span>

  <button
    type="button"
    class="valve-designation-dropdown__button"
    @click="
      isValveDesignationDropdownOpen =
        !isValveDesignationDropdownOpen
    "
  >
    <span>
  {{
    selectedValveDesignationKey
      ? (
          getValveDesignationOptions()
            .find(
              (option) =>
                option.key ===
                selectedValveDesignationKey
            )?.label ??
          'Nenhuma válvula'
        )
      : 'Nenhuma válvula'
  }}
</span>

    <span
      class="valve-designation-dropdown__arrow"
    >
      {{
        isValveDesignationDropdownOpen
          ? '▲'
          : '▼'
      }}
    </span>
  </button>

  <div
    v-if="
      isValveDesignationDropdownOpen
    "
    class="valve-designation-dropdown__menu"
  >
  <button
  type="button"
  :class="[
    'valve-designation-dropdown__option',
    {
      'valve-designation-dropdown__option--selected':
        !selectedValveDesignationKey
    }
  ]"
  @click="
    selectValveDesignationOption('')
  "
>
  Nenhuma válvula
</button>
    <button
      v-for="
        option in
        getValveDesignationOptions()
      "
      :key="option.key"
      type="button"
      :class="[
        'valve-designation-dropdown__option',
        {
          'valve-designation-dropdown__option--selected':
            option.key ===
            selectedValveDesignationKey
        }
      ]"
      @click="
        selectValveDesignationOption(
          option.key
        )
      "
    >
      {{ option.label }}
    </button>
  </div>
</div>

<div
  v-if="selectedValveDesignationKey"
  class="selected-valve-summary"
>
  <p>
    <strong>
      Estado atual:
    </strong>

    {{ getSelectedValveStateLabel() }}
  </p>

  <p>
  <strong>
    Tipo original:
  </strong>

  {{ getSelectedValveNormalTypeLabel() }}
</p>

<p>
  <strong>
    Tubos controlados:
  </strong>

  {{
    getManualValveControlledPipeCount()
  }}
</p>

  <p>
    <strong>
      Nome original:
    </strong>

    {{
      selectedValveOriginalDesignation ||
      'não disponível'
    }}
  </p>
</div>

<div
  v-if="selectedValveDesignationKey"
  class="selected-valve-control-actions"
>
  <button
    type="button"
    :disabled="
      getSelectedValveStateLabel() ===
      'aberta'
    "
    @click="
      setSelectedValvesState(
        'open'
      )
    "
  >
    Abrir válvula
  </button>

  <button
    type="button"
    :disabled="
      getSelectedValveStateLabel() ===
      'fechada'
    "
    @click="
      setSelectedValvesState(
        'closed'
      )
    "
  >
    Fechar válvula
  </button>
</div>

<div class="flow-section-title">
  Tubos controlados
</div>

<p
  v-if="!selectedValveDesignationKey"
  class="connection-note"
>
  Escolhe uma válvula no campo acima para
  consultar ou definir os tubos controlados.
</p>

<div
  v-if="selectedValveDesignationKey"
  class="
    manual-valve-pipe-definition
  "
>
  <p
    class="
      connection-note
      workflow-help-note
    "
  >
    Define manualmente os tubos cujas
    setas devem desaparecer quando a
    válvula estiver fechada.
  </p>

  <div
    class="
      manual-valve-pipe-definition__summary
    "
  >

    <p>
      <strong>
        Tubos controlados:
      </strong>

      {{
        getManualValveControlledPipeCount()
      }}
    </p>

    <p
      v-if="
        isManualValvePipeDefinitionMode
      "
    >
      <strong>
        Tubos no rascunho:
      </strong>

      {{
        manualValvePipeDraft.length
      }}
    </p>
  </div>

  <div
  v-if="
    !isManualValvePipeDefinitionMode
  "
  class="
    manual-valve-pipe-definition__saved-actions
  "
>
  <button
    type="button"
    :disabled="
      !selectedValveDesignationKey
    "
    @click="
      startManualValvePipeDefinition
    "
  >
    Definir tubos controlados
  </button>

  <button
    type="button"
    :disabled="
      !selectedValveDesignationKey ||
      getManualValveControlledPipeCount() ===
        0
    "
    @click="
      highlightSavedManualValvePipes
    "
  >
    Realçar tubos controlados
  </button>

  <button
    type="button"
    class="flow-button--danger"
    :disabled="
      !selectedValveDesignationKey ||
      getManualValveControlledPipeCount() ===
        0
    "
    @click="
      removeManualValvePipeDefinition
    "
  >
    Remover definição
  </button>
</div>

  <div
    v-else
    class="
      manual-valve-pipe-definition__active
    "
  >
    <p class="manual-route-status">
      Modo ativo: seleciona no modelo os
      tubos controlados por esta válvula.
    </p>

    <p class="connection-note">
  Selecionados no modelo:
  <strong>
    {{ selectedCount }}
  </strong>
</p>

<div
  class="
    flow-actions
    flow-actions--secondary
  "
>
  <button
    type="button"
    :disabled="
      selectedCount === 0
    "
    @click="
      addSelectedPipesToManualValveDraft
    "
  >
    Adicionar selecionados
  </button>

  <button
    type="button"
    :disabled="
      selectedCount === 0 ||
      manualValvePipeDraft.length === 0
    "
    @click="
      removeSelectedPipesFromManualValveDraft
    "
  >
    Retirar selecionados
  </button>

  <button
    type="button"
    :disabled="
      manualValvePipeDraft.length === 0
    "
    @click="
      clearManualValvePipeDraft
    "
  >
    Limpar rascunho
  </button>
</div>

<p
  v-if="
    hasManualValvePipeDraftChanges
  "
  class="connection-note workflow-help-note"
>
  Existem alterações no rascunho que
  ainda não foram guardadas.
</p>

<div
  class="
    flow-actions
    flow-actions--secondary
  "
>
  <button
    type="button"
    class="flow-button--primary"
    :disabled="
      manualValvePipeDraft.length ===
      0
    "
    @click="
      saveManualValvePipeDefinition
    "
  >
    Guardar definição
  </button>

  <button
    type="button"
    :disabled="
      manualValvePipeDraft.length ===
      0
    "
    @click="
      highlightManualValvePipeDraft
    "
  >
    Realçar rascunho
  </button>
</div>

<div
  class="
    flow-actions
    flow-actions--single
  "
>
  <button
    type="button"
    class="flow-button--danger"
    @click="
      cancelManualValvePipeDefinition
    "
  >
    Cancelar definição
  </button>
</div>

</div>

</div>

</div>

<div
  class="
    flow-section-title
    flow-section-title--button
  "
>
  <span>
    Classificar válvulas
  </span>

  <button
    type="button"
    class="section-collapse-button"
    @click="
      isValveClassificationPanelOpen =
        !isValveClassificationPanelOpen
    "
  >
    {{
      isValveClassificationPanelOpen
        ? '−'
        : '+'
    }}
  </button>
</div>

<div
  v-if="isValveClassificationPanelOpen"
>
  <p class="selection-count">
    Elementos selecionados:
    {{ selectedCount }}
  </p>

  <div class="selected-mep-info">
    {{ selectedMepElementInfo }}
  </div>

  <p class="connection-note">
    Seleciona uma válvula no modelo e define
    se é normalmente aberta ou normalmente
    fechada. Esta indicação é apenas informativa.
  </p>

  <div
    class="
      flow-actions
      flow-actions--secondary
    "
  >
    <button
      type="button"
      :disabled="
        selectedCount === 0
      "
      @click="
        defineSelectedElementsAs(
          'normallyOpenValve'
        )
      "
    >
      Definir como válvula NA
    </button>

    <button
      type="button"
      :disabled="
        selectedCount === 0
      "
      @click="
        defineSelectedElementsAs(
          'normallyClosedValve'
        )
      "
    >
      Definir como válvula NF
    </button>
  </div>

  <div
    class="
      flow-actions
      flow-actions--single
    "
  >
    <button
      type="button"
      class="flow-button--danger"
      :disabled="
        selectedCount === 0
      "
      @click="
        deleteSelectedElementDefinitions
      "
    >
      Apagar definição selecionada
    </button>
  </div>

  <dl class="flow-stats">
    <div>
      <dt>Válvulas NA</dt>

      <dd>
        {{
          countMepElementsByType(
            'normallyOpenValve'
          )
        }}
      </dd>
    </div>

    <div>
      <dt>Válvulas NF</dt>

      <dd>
        {{
          countMepElementsByType(
            'normallyClosedValve'
          )
        }}
      </dd>
    </div>
  </dl>

  <div class="flow-section-title">
  Realçar válvulas por tipo
</div>

<div
  class="
    flow-actions
    flow-actions--secondary
  "
>
  <button
    type="button"
    @click="
      highlightMepElementsByType(
        'normallyOpenValve'
      )
    "
  >
    Realçar válvulas NA
  </button>

  <button
    type="button"
    @click="
      highlightMepElementsByType(
        'normallyClosedValve'
      )
    "
  >
    Realçar válvulas NF
  </button>
</div>

<div
  class="
    flow-actions
    flow-actions--single
  "
>
  <button
    type="button"
    @click="
      clearValveTypeHighlight
    "
  >
    Limpar realce
  </button>
</div>
</div>

    <div
    class="flow-section-title flow-section-title--button"
  >
    <span>
  Gerir várias válvulas
</span>

    <button
      type="button"
      class="section-collapse-button"
      @click="toggleValveManagementPanel"
    >
      {{
        isValveManagementPanelOpen
          ? '−'
          : '+'
      }}
    </button>
  </div>

  <div
    v-if="isValveManagementPanelOpen"
    class="valve-management-panel"
  >
    <p class="connection-note">
      Válvulas detetadas:
      {{ getValveManagementOptions().length }}
    </p>

    <p
      v-if="
        !getValveManagementOptions().length
      "
      class="connection-note workflow-help-note"
    >
      Nenhuma válvula detetada. Faz novamente
      o scan do IFC.
    </p>

    <div
      v-else
      class="valve-management-list"
    >
      <label
        v-for="
          valveOption in
          getValveManagementOptions()
        "
        :key="
          'valve-management-' +
          valveOption.key
        "
        class="valve-management-item"
      >
        <input
          type="checkbox"
          :checked="
            isValveSelectedForManagement(
              valveOption.key
            )
          "
          @change="
            toggleValveManagementSelection(
              valveOption.key
            )
          "
        />

        <span>
          {{ valveOption.label }}
        </span>
      </label>
    </div>

    <div
  class="flow-actions flow-actions--secondary"
>
  <button
    type="button"
    :disabled="
      getValveManagementOptions().length === 0 ||
      areAllManagedValvesSelected()
    "
    @click="
      selectAllManagedValves
    "
  >
    Selecionar todas
  </button>

  <button
    type="button"
    :disabled="
      selectedValveKeysForManagement.size === 0
    "
    @click="
      clearValveManagementSelection
    "
  >
    Desmarcar todas
  </button>
</div>

    <p class="connection-note">
      Selecionadas:
      {{
        selectedValveKeysForManagement.size
      }}
    </p>

    <p class="connection-note">
      Seleciona uma ou várias válvulas e escolhe
      uma das ações seguintes.
    </p>

    <div
      class="flow-actions flow-actions--secondary"
    >
      <button
        type="button"
        :disabled="
          selectedValveKeysForManagement.size === 0
        "
        @click="
          highlightSelectedManagedValves
        "
      >
        Realçar selecionadas
      </button>

      <button
        type="button"
        :disabled="
          highlightedValveKeysForManagement.size === 0
        "
        @click="
          clearManagedValveHighlight
        "
      >
        Limpar realce
      </button>
    </div>

<div
  class="flow-actions flow-actions--secondary"
>
  <button
    type="button"
    :disabled="
      selectedValveKeysForManagement.size === 0
    "
    @click="
      setSelectedManagedValvesState(
        'open'
      )
    "
  >
    Abrir selecionadas
  </button>

  <button
    type="button"
    :disabled="
      selectedValveKeysForManagement.size === 0
    "
    @click="
      setSelectedManagedValvesState(
        'closed'
      )
    "
  >
    Fechar selecionadas
  </button>
</div>

<p class="connection-note">
  Abre ou fecha manualmente as válvulas
  selecionadas. A indicação NA/NF é apenas
  informativa.
</p>

<div
  class="
    flow-actions
    flow-actions--secondary
  "
>
  <button
    type="button"
    :disabled="
      selectedValveKeysForManagement.size ===
      0
    "
    @click="
      clearValveManagementSelection
    "
  >
    Limpar seleção
  </button>

  <button
    type="button"
    class="flow-button--danger"
    :disabled="
      selectedValveKeysForManagement.size ===
      0
    "
    @click="
      excludeSelectedManagedValves
    "
  >
    Eliminar selecionadas
  </button>
</div>

    <p class="connection-note workflow-help-note">
      As válvulas automáticas eliminadas voltarão
      a aparecer quando repetires o scan do IFC.
      As válvulas manuais terão de ser novamente
      definidas manualmente.
    </p>
  </div>

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

<div
  v-if="
    false &&
    isValveAssociationDetailsOpen
  "
>
  <div class="valve-association-summary">
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

<p class="connection-note workflow-help-note">
  Uma válvula associada bloqueia os tubos
  controlados quando está fechada e volta a
  permitir o fluxo quando é aberta.
</p>

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

<div class="flow-actions flow-actions--single">
  <button
  type="button"
  :disabled="
    isPreparingAutomaticValveAssociations
  "
  @click="
    handleAutomaticValveAssociationClick
  "
>
  {{
    isPreparingAutomaticValveAssociations
      ? 'A calcular ' +
        automaticValveAssociationProgress +
        ' de ' +
        automaticValveAssociationTotal +
        '...'
      : 'Associar automaticamente todas'
  }}
</button>

<div
  v-if="
    isPreparingAutomaticValveAssociations
  "
  class="automatic-valve-calculation-status"
>
  <div
    class="
      automatic-valve-calculation-status__header
    "
  >
    <span>
      A calcular associações automáticas
    </span>

    <strong>
      {{
        automaticValveAssociationProgress
      }}
      /
      {{
        automaticValveAssociationTotal
      }}
    </strong>
  </div>

  <div
    class="
      automatic-valve-calculation-progress
    "
  >
    <div
      class="
        automatic-valve-calculation-progress__fill
      "
      :style="{
        width:
          (
            automaticValveAssociationTotal > 0
              ? (
                  automaticValveAssociationProgress /
                  automaticValveAssociationTotal
                ) * 100
              : 0
          ) + '%'
      }"
    ></div>
  </div>

  <p class="connection-note">
    Aguarda enquanto as válvulas são comparadas
    com os caminhos protegidos.
  </p>
</div>
</div>

<p class="connection-note">
  Serão processadas apenas as válvulas de
  corte que ainda não tenham associação.
  Os casos ambíguos serão ignorados para
  revisão manual.
</p>

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
</div>
      </section>
    
<section
  v-if="
    activeApplicationTab === 'manual'
  "
  :class="[
    'flow-panel',
    'manual-reset-section',
    {
      'flow-panel--minimized':
        isResetPanelMinimized
    }
  ]"
  aria-label="
    Limpeza e reposição da configuração
  "
>
  <div class="flow-panel__header">
    <div>
      <p class="flow-panel__eyebrow">
        Configuração
      </p>

      <h2>
        Limpeza e reposição
      </h2>
    </div>

    <button
      type="button"
      class="flow-panel__toggle"
      @click="
        toggleResetPanelMinimized
      "
    >
      {{
        isResetPanelMinimized
          ? '+'
          : '−'
      }}
    </button>
  </div>

  <div
    v-if="
      !isResetPanelMinimized
    "
    class="flow-panel__content"
  >

  <div class="configuration-reset-option">
    <strong>
      Repor alterações manuais
    </strong>

    <p>
      Mantém os caminhos, ciclos e circuitos.
      Remove sentidos corrigidos, cores
      personalizadas, uniões, proteções,
      associações e restantes alterações
      manuais.
    </p>

    <p class="connection-note">
      As válvulas regressam ao estado original.
    </p>

    <button
      type="button"
      class="
        configuration-reset-button
        configuration-reset-button--secondary
      "
      @click="
        resetManualChangesOnly
      "
    >
      Repor alterações manuais
    </button>
  </div>

  <div
    class="
      configuration-reset-option
      configuration-reset-option--danger
    "
  >
    <strong>
      Começar completamente de novo
    </strong>

    <p>
      Apaga todos os dados guardados para o
      IFC atual, incluindo caminhos, ciclos,
      circuitos, válvulas e configurações.
      O ficheiro IFC original não será apagado.
    </p>

    <button
      type="button"
      class="
        configuration-reset-button
        configuration-reset-button--danger
      "
      @click="
        resetAllManualConfiguration
      "
    >
      Apagar tudo e começar de novo
    </button>
  </div>

  </div>
</section>

    <section
  v-if="activeApplicationTab === 'simulation'"
  :class="[
    'flow-panel',
    isSimulationControlPanelMinimized
      ? 'flow-panel--minimized'
      : ''
  ]"
  aria-label="Simulação da central"
>
  <div class="flow-panel__header">
    <div>
      <p class="flow-panel__eyebrow">Simulação</p>
      <h2>Controlo da central</h2>
    </div>

    <span
  :class="[
    'flow-status',
    isFlowing ? 'flow-status--on' : ''
  ]"
>
  {{ isFlowing ? 'ON' : 'OFF' }}
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

  <div
  class="
    flow-section-title
    flow-section-title--button
  "
>
  <span>
    Ciclos visíveis
  </span>

  <button
    type="button"
    class="section-collapse-button"
    @click="
      isSimulationCyclesSectionOpen =
        !isSimulationCyclesSectionOpen
    "
  >
    {{
      isSimulationCyclesSectionOpen
        ? '−'
        : '+'
    }}
  </button>
</div>

<div v-if="isSimulationCyclesSectionOpen">
  <div class="simulation-cycle-selection">
    <label
      v-for="
        cycleNumber in waterCycleCount
      "
      :key="
        'simulation-cycle-' +
        cycleNumber
      "
      class="simulation-cycle-option"
    >
      <input
        type="checkbox"
        :checked="
          isSimulationCycleSelected(
            cycleNumber
          )
        "
        @change="
          toggleSimulationCycleSelection(
            cycleNumber
          )
        "
      />

      <span>
        {{
          getCycleDisplayName(
            cycleNumber
          )
        }}
      </span>
    </label>
  </div>

  <div
    class="
      flow-actions
      flow-actions--secondary
    "
  >
    <button
      type="button"
      @click="
        selectAllSimulationCycles
      "
    >
      Todos
    </button>

    <button
      type="button"
      @click="
        clearSimulationCycleSelection
      "
    >
      Nenhum
    </button>
  </div>

  <div
    class="
      flow-actions
      flow-actions--single
    "
  >
    <button
      type="button"
      :disabled="
        selectedSimulationCycles.size ===
        0
      "
      :class="{
        'cycle-view-button--active':
          isCycleViewFilterActive
      }"
      @click="
        applySimulationCycleFilter
      "
    >
      Aplicar visualização
    </button>
  </div>

  <p
    v-if="isCycleViewFilterActive"
    class="
      automatic-analysis-status
      automatic-analysis-status--ready
    "
  >
    Ciclos visíveis:
    {{
      getSelectedSimulationCyclesLabel()
    }}
  </p>

  <p
    v-else
    class="connection-note"
  >
    Todos os ciclos estão visíveis.
  </p>
</div>

<div
  class="
    flow-section-title
    flow-section-title--button
  "
>
  <span>
    Animação
  </span>

  <button
    type="button"
    class="section-collapse-button"
    @click="
      isSimulationAnimationSectionOpen =
        !isSimulationAnimationSectionOpen
    "
  >
    {{
      isSimulationAnimationSectionOpen
        ? '−'
        : '+'
    }}
  </button>
</div>

<div
  v-if="
    isSimulationAnimationSectionOpen
  "
>
  <div
    class="
      flow-actions
      flow-actions--secondary
    "
  >
    <button
      type="button"
      :disabled="
        isPreparingFlowAnimation
      "
      @click="
        updateFlowAnimationWithoutStarting
      "
    >
      {{
        isPreparingFlowAnimation
          ? 'A atualizar...'
          : 'Atualizar'
      }}
    </button>

    <button
      type="button"
      :disabled="
        isPreparingFlowAnimation
      "
      @click="
        toggleFlow
      "
    >
      {{
        isPreparingFlowAnimation
          ? 'A preparar...'
          : isFlowing
            ? 'Pausar simulação'
            : 'Iniciar simulação'
      }}
    </button>
  </div>

  <div
    v-if="
      isPreparingFlowAnimation ||
      isFlowAnimationReady ||
      hasFlowPreparationError
    "
    class="flow-preparation-status"
  >
    <div
      class="
        flow-preparation-status__header
      "
    >
      <span>
        {{
          hasFlowPreparationError
            ? 'Erro na preparação'
            : isPreparingFlowAnimation ||
                flowPreparationProgress <
                  100
              ? 'A preparar animação...'
              : 'Animação pronta'
        }}
      </span>

      <strong>
        {{ flowPreparationProgress }}%
      </strong>
    </div>

    <div
      class="flow-preparation-progress"
      role="progressbar"
      aria-label="Preparação da animação"
      aria-valuemin="0"
      aria-valuemax="100"
      :aria-valuenow="
        flowPreparationProgress
      "
    >
      <div
        class="
          flow-preparation-progress__fill
        "
        :class="{
          'flow-preparation-progress__fill--ready':
            isFlowAnimationReady &&
            flowPreparationProgress ===
              100 &&
            !hasFlowPreparationError,

          'flow-preparation-progress__fill--error':
            hasFlowPreparationError
        }"
        :style="{
          width:
            flowPreparationProgress +
            '%'
        }"
      ></div>
    </div>
  </div>

  <label class="flow-slider">
    <span>
      Velocidade
    </span>

    <input
      v-model.number="flowSpeed"
      type="range"
      min="0.2"
      max="3"
      step="0.1"
    />
  </label>
</div>

    <p class="connection-note">
  Percursos guardados: {{ savedRoutes.length }}
</p>

    <p class="connection-note">
      Tubos marcados: {{ pipeStats.total }}
    </p>

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
    <span>Percursos guardados</span>
    <strong>{{ savedRoutes.length }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Percursos visíveis</span>
    <strong>{{ countVisibleSavedRoutes() }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Percursos ocultos</span>
    <strong>{{ countHiddenSavedRoutes() }}</strong>
  </div>

  <div class="central-summary__item">
    <span>Percursos protegidos</span>
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

    <p class="flow-note">
      {{ flowMessage }}
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
import * as WEBIFC from "web-ifc";
import * as THREE from "three";
import * as OBC from "@thatopen/components";
import * as FRAGS from "@thatopen/fragments";
import * as BUI from "@thatopen/ui";
import * as BUIC from "@thatopen/ui-obc";
import * as OBCF from "@thatopen/components-front";

type ApplicationTab =
  | "automatic"
  | "manual"
  | "simulation";
type PipeCircuit = string;
type RouteCreationMethod =
  | "automatic"
  | "manual";
const UNASSIGNED_ROUTE_CIRCUIT =
  "unassigned-route";
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
  systemName?: string;
systemType?: string;
ifcNormalState?: string;
isShutoffValve?: boolean;
valveIdentificationText?: string;
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

type ManualValveControlledPipe = {
  modelId: string;
  localId: number;
};

type ProtectedRouteDirection = {
  node: FlowNode;
  previous: FlowNode | null;
  next: FlowNode | null;
  reversed: boolean;
};

type ProtectedRouteSnapshot = {
  name: string;
  temperature:
    PipeCircuit;
  path:
    FlowNode[];
  hidden:
    boolean;
  groupId:
    string | null;
  groupName:
    string;
  effectiveColor:
    string;
  circuitLabel:
    string;
  originalCircuitColor:
    string | null;
  customColor:
    string | null;
  directionStarts:
    FlowNode[];
  directionEnds:
    FlowNode[];
  needsDirectionRedefinition:
    boolean;
  protectedDirections:
    ProtectedRouteDirection[];
};

type SavedRouteMergeSource = {
  id: string;
  name: string;
  temperature: PipeCircuit;
  path: FlowNode[];
  hidden?: boolean;
  locked?: boolean;
  groupId?: string;
  originalCircuitColor?: string;
  customColor?: string;
  directionStarts?: FlowNode[];
  directionEnds?: FlowNode[];
  needsDirectionRedefinition?: boolean;
  protectedDirections?: ProtectedRouteDirection[];
  mergeBackup?: SavedRouteMergeBackup;
  protectedSnapshot?:
  ProtectedRouteSnapshot;
};

type SavedRouteMergeBackup = {
  sourceRoutes: SavedRouteMergeSource[];

  sourceValveLinks: Array<{
    valveKey: string;
    linkedPipes: ValveControlledPipeLink[];
  }>;

  sourceBlockedValveLinks: Array<{
    valveKey: string;
    linkedPipes: ValveControlledPipeLink[];
  }>;

  sourceSimulationRouteIds: string[];
};

type SavedRoute = {
  id: string;
  name: string;
  temperature: PipeCircuit;
  path: FlowNode[];
  hidden?: boolean;
  locked?: boolean;
  groupId?: string;
  originalCircuitColor?: string;
  customColor?: string;
  directionStarts?: FlowNode[];
  directionEnds?: FlowNode[];
  needsDirectionRedefinition?: boolean;
  protectedDirections?: ProtectedRouteDirection[];
  mergeBackup?: SavedRouteMergeBackup;
  protectedSnapshot?:
  ProtectedRouteSnapshot;
};

type SavedRouteGroup = {
  id: string;
  name: string;
  color: string;
};

type RouteComparisonResult = {
  firstRoute: SavedRoute;
  secondRoute: SavedRoute;
  firstOnlyNodes: string[];
  secondOnlyNodes: string[];
  commonNodeCount: number;
  overlapPercentage: number;
  exactDuplicate: boolean;
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

type AutomaticValveRouteMatch = {
  route: SavedRoute;
  closestNode: FlowNode;
  distance: number;
  ambiguous: boolean;
  alternativeRouteName?: string;
  alternativeDistance?: number;
};

type AutomaticValveAssociationPreview = {
  valveKey: string;

  valveNode: FlowNode;

  valveLabel: string;

  normalStateLabel: string;

  currentStateLabel: string;

  routeId: string | null;

  routeName: string;

  closestNode: FlowNode | null;

  distance: number | null;

  downstreamPipeCount: number;

  status:
    | "safe"
    | "noMatch"
    | "noDownstream"
    | "ambiguous";

  accepted: boolean;
};

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

type PipeGeometryAxis = {
  start: any;
  end: any;
  center: any;
  length: number;
};

type PipeGraphItem = FlowNode & {
  box: any;
  center: any;
  endpoints: any[];
};

const activeApplicationTab =
  ref<ApplicationTab>("automatic");
const containerRef = ref<HTMLDivElement | null>(null);
const ifcInput = ref<HTMLInputElement | null>(null);
const isLoading = ref(false);
const isFlowing = ref(false);
const isCentralSimulationRunning = ref(false);
const isManualFlowAnimationRunning = ref(false);
const isFlowManuallyPaused = ref(false);
const loadingProgress = ref(0);
const loadingFileName = ref("");
const flowSpeed = ref(0.2);
const isPreparingFlowAnimation =
  ref(false);
const flowPreparationProgress =
  ref(0);
const isFlowAnimationReady =
  ref(false);
const hasFlowPreparationError =
  ref(false);
let flowPreparationRunId = 0;
const flowMessage = ref("Seleciona tubos no modelo e atribui um circuito.");
const discardRouteMessage = ref("");
const waterCycleCount = ref(3);
const pendingWaterCycleCount = ref(3);
const cycleNames = reactive<Record<string, string>>({});
const activeCycleNumber = ref(1);
const selectedSimulationCycles =
  reactive<Set<number>>(new Set());
const isCycleViewFilterActive =
  ref(false);
const targetCycleNumberForSavedRoutes = ref(1);
const cycleCircuitDefinitions = reactive<CycleCircuitDefinition[]>([]);
const isCycleCircuitPanelOpen = ref(false);
const pendingCycleCircuitKind = ref<CycleCircuitKind>("extra");
const pendingCycleCircuitName = ref("");
const pendingCycleCircuitColor = ref("#2e7d32");
const selectedCycleCircuitKey = ref("");
const isIfcPanelCollapsed = ref(false);
const isCycleConfigurationSectionOpen =
  ref(false);
const isSelectedPipeActionsSectionOpen =
  ref(false);
const isCycleNamesPanelOpen = ref(false);
const isSavedRoutesPanelOpen = ref(true);
const savedRouteSearchText =
  ref("");
const isPathStatsPanelOpen = ref(false);
const highlightedSavedRouteId = ref<string | null>(null);
const isApplyingSavedRouteHighlight = ref(false);
const isSavedRouteDirectionPanelOpen =
  ref(false);
const isRouteDirectionFocusModeActive =
  ref(false);
const selectedSavedRouteDirectionId =
  ref("");
const savedRouteDirectionStarts =
  ref<FlowNode[]>([]);
const savedRouteDirectionEnds =
  ref<FlowNode[]>([]);
const savedRouteDirectionWarning =
  ref("");
const isSimulationCyclesSectionOpen =
  ref(false);
const isSimulationAnimationSectionOpen =
  ref(true);
const isCentralSummaryOpen = ref(false);
const selectedCount = ref(0);
const selectedMepElementInfo = ref("Nenhum elemento classificado selecionado.");
const selectedTubeRouteInfo =
  ref("");
const selectedIfcDetailsText = ref("");
const isIfcDetailsPanelOpen = ref(false);
const isAutomaticAnalysisRunning = ref(false);
const hasAutomaticAnalysisResults = ref(false);
const isSystemScanRunning = ref(false);
const hasSystemScanResults = ref(false);
const systemScanResults = reactive({
  systemTypes: [] as string[],
  systemNames: [] as string[],
});
const pipeTypeFlowNodes = new Set<string>();
const ignoredAutomaticPathNodes = new Map<
  string,
  Set<number>
>();
const scannedSystemGroups = new Map<
  string,
  {
    nodes: Array<
      FlowNode & {
        revitElementId: number;
      }
    >;
    systemTypes: Set<string>;
  }
>();
const automaticOrderedCircuitNodes =
  new Map<PipeCircuit, FlowNode[]>();

function automaticDirectionKey(
  circuit: PipeCircuit,
  node: FlowNode,
) {
  return circuit + "|" + nodeKey(node);
}

const automaticDirectionNeighbors =
  new Map<
    string,
    {
      previous: FlowNode | null;
      next: FlowNode | null;
    }
  >();
const isSystemTypesListOpen = ref(false);
const isSystemNamesListOpen = ref(false);
const automaticAnalysisResults = reactive({
  pipes: 0,
  valves: 0,
  equipment: 0,
  total: 0,
});
const selectedValveDesignation = ref("nenhuma válvula selecionada");
const pendingValveDesignation = ref("");
const selectedValveOriginalDesignation = ref("");
const selectedValveDesignationKey = ref("");
const isValveDesignationDropdownOpen =
  ref(false);
const highlightedValveFromDropdown = ref<FlowNode | null>(null);
const isValveFocusModeActive =
  ref(false);
const isValveDesignationPanelOpen = ref(false);
const isValveClassificationPanelOpen =
  ref(false);
const isValveRenamePanelOpen = ref(false);
const isValveAssociationDetailsOpen = ref(true);
const valveOriginalDesignations =
  reactive<Record<string, string>>({});
const excludedValveKeys =
  reactive<Set<string>>(
    new Set(),
  );
const selectedValveKeysForManagement =
  reactive<Set<string>>(
    new Set(),
  );
const highlightedValveKeysForManagement =
  reactive<Set<string>>(
    new Set(),
  );
const isValveManagementPanelOpen =
  ref(false);
const automaticValveAssociationPreviews =
  reactive<
    AutomaticValveAssociationPreview[]
  >([]);
const isAutomaticValveAssociationPreviewOpen =
  ref(false);
const isPreparingAutomaticValveAssociations =
  ref(false);
const automaticValveAssociationProgress =
  ref(0);
const automaticValveAssociationTotal =
  ref(0);
const hasLoadedModel = ref(false);
const activeIfcStorageId = ref("");
const isElementPanelMinimized = ref(true);
const isIfcInformationPanelMinimized =
  ref(true);
const isFlowControlsPanelMinimized = ref(true);
const isSimulationControlPanelMinimized = ref(true);
const isResetPanelMinimized =
  ref(true);
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
const ROUTE_GROUPS_STORAGE_KEY =
  "bastto-viewer-route-groups";
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
const PIPE_TYPE_FLOW_NODES_STORAGE_KEY =
  "bastto-viewer-pipe-type-flow-nodes";
const SYNCED_PIPE_DIRECTIONS_STORAGE_KEY =
  "bastto-viewer-synced-pipe-directions";
const VALVE_PIPE_LINKS_STORAGE_KEY =
  "bastto-viewer-valve-pipe-links";
const MANUAL_VALVE_CONTROLLED_PIPES_STORAGE_KEY =
  "bastto-viewer-manual-valve-controlled-pipes";
const EXCLUDED_VALVES_STORAGE_KEY =
  "bastto-viewer-excluded-valves";

function getIfcStorageKey(
  baseKey: string,
) {
  if (!activeIfcStorageId.value) {
    return "";
  }

  return (
    baseKey +
    "::" +
    activeIfcStorageId.value
  );
}

function getActiveIfcStorageItem(
  baseKey: string,
) {
  const storageKey =
    getIfcStorageKey(baseKey);

  if (!storageKey) {
    return null;
  }

  return localStorage.getItem(
    storageKey,
  );
}

function setActiveIfcStorageItem(
  baseKey: string,
  value: string,
) {
  const storageKey =
    getIfcStorageKey(baseKey);

  if (!storageKey) {
    return;
  }

  localStorage.setItem(
    storageKey,
    value,
  );
}

function removeActiveIfcStorageItem(
  baseKey: string,
) {
  const storageKey =
    getIfcStorageKey(baseKey);

  if (!storageKey) {
    return;
  }

  localStorage.removeItem(
    storageKey,
  );
}

function createIfcStorageId(
  file: File,
) {
  return [
    file.name
      .trim()
      .toLowerCase(),

    file.size,

    file.lastModified,
  ].join("|");
}

function clearActiveModelConfigurationFromMemory() {
  clearFlowLayer();

  savedRoutes.splice(0);
  savedRouteGroups.splice(0);
  cycleCircuitDefinitions.splice(0);
  flowConnections.splice(0);
  currentRouteConnections.splice(0);
  routeWaypoints.splice(0);
  manualRouteNodes.splice(0);

  for (
    const key of
      Object.keys(mepElements)
  ) {
    delete mepElements[key];
  }

  for (
    const key of
      Object.keys(cycleNames)
  ) {
    delete cycleNames[key];
  }

  for (
    const key of
      Object.keys(manualAssignments)
  ) {
    delete manualAssignments[key];
  }

  selectedRouteIdsForGrouping.clear();

selectedRouteIdsForSimulation.clear();

selectedSavedRouteIdForEditing.value =
  "";

automaticValveAssociationPreviews.splice(
  0,
);

automaticValveAssociationProgress.value =
  0;

automaticValveAssociationTotal.value =
  0;

isAutomaticValveAssociationPreviewOpen.value =
  false;

isPreparingAutomaticValveAssociations.value =
  false;

  reversedPipeDirections.clear();
  syncedPipeDirections.clear();
  hiddenFlowArrowElements.clear();
  pipeTypeFlowNodes.clear();

  automaticDirectionNeighbors.clear();
  automaticOrderedCircuitNodes.clear();

  blockedPipes.clear();
  blockedRoutePipes.clear();

  valveBlockedPipeLinks.clear();
  valveControlledPipeLinks.clear();
  excludedValveKeys.clear();
  selectedValveKeysForManagement.clear();

  selectedValveDesignation.value =
    "nenhuma válvula selecionada";

  selectedValveOriginalDesignation.value =
    "";

  selectedValveDesignationKey.value =
    "";

  pendingValveDesignation.value =
    "";

  selectedValveForPipeLink.value =
    null;

  highlightedValveFromDropdown.value =
    null;

  selectedValveAssociationRouteId.value =
    "";

  highlightedSavedRouteId.value =
    null;

  selectedSavedRouteDirectionId.value =
    "";

  savedRouteDirectionStarts.value =
    [];

  savedRouteDirectionEnds.value =
    [];

  selectedCycleCircuitKey.value =
    "";

  routeStart = null;
  routeEnd = null;

  routeStartLabel.value =
    "nenhum";

  routeEndLabel.value =
    "nenhum";

  blockedCount.value = 0;

  pipeStats.supply = 0;
  pipeStats.return = 0;
  pipeStats.total = 0;

  waterCycleCount.value = 3;
  pendingWaterCycleCount.value = 3;

  activeCycleNumber.value = 1;

  hasSystemScanResults.value =
    false;

  systemScanResults.systemTypes.splice(
    0,
  );

  systemScanResults.systemNames.splice(
    0,
  );

  scannedSystemGroups.clear();

  hasAutomaticAnalysisResults.value =
    false;

  automaticAnalysisResults.pipes = 0;
  automaticAnalysisResults.valves = 0;
  automaticAnalysisResults.equipment = 0;
  automaticAnalysisResults.total = 0;

  isFlowing.value = false;

  isFlowAnimationReady.value =
    false;

  isPreparingFlowAnimation.value =
    false;

  flowPreparationProgress.value = 0;

  hasFlowPreparationError.value =
    false;
}

function loadActiveIfcConfiguration() {
  if (!activeIfcStorageId.value) {
    return;
  }

  loadWaterCycleCountFromStorage();
  loadCycleNamesFromStorage();

  loadCycleCircuitDefinitionsFromStorage();

  loadMepElementsFromStorage();
  loadRoutesFromStorage();
  loadRouteGroupsFromStorage();

  loadReversedDirectionsFromStorage();
  loadSyncedPipeDirectionsFromStorage();

  loadHiddenFlowArrowsFromStorage();
  loadPipeTypeFlowNodesFromStorage();

  loadValvePipeLinksFromStorage();
  loadManualValveControlledPipesFromStorage();
  loadExcludedValvesFromStorage();
  ensureConfiguredAssignments();
  ensureCycleNames();
  ensureCycleCircuitDefinitions();

  updateManualStats();
}

let world: any;
let serializer: FRAGS.IfcImporter;
let fragmentManager: OBC.FragmentsManager;
let modelHighlighter: OBCF.Highlighter;

const persistentCircuitHighlightStyles =
  new Set<string>();
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
const selectedSavedRouteIdForEditing =
  ref("");
const isCreateRouteSectionOpen =
  ref(false);
const newRouteCycleSelection =
  ref("");
const newRouteCircuitKind =
  ref<CycleCircuitKind>(
    "hotSupply",
  );
const newRouteCircuitName =
  ref("Ida quente");
const newRouteCircuitColor =
  ref("#ff0000");
const newRouteDraftCircuitKey =
  ref("");
const routeCreationMethod =
  ref<RouteCreationMethod>(
    "automatic",
  );
const isEditRouteSectionOpen =
  ref(false);
const mepElements = reactive<Record<string, MepElement>>({});
const savedRoutes = reactive<SavedRoute[]>([]);
const savedRouteGroups =
  reactive<SavedRouteGroup[]>([]);
const selectedRouteIdsForSimulation =
  reactive<Set<string>>(
    new Set(),
  );
const selectedRouteIdsForGrouping =
  reactive<Set<string>>(new Set());
const isRouteGroupDialogOpen = ref(false);
const editingRouteGroupId =
  ref<string | null>(null);
const pendingRouteGroupName = ref(
  "Novo grupo de caminhos",
);
const pendingRouteGroupColor = ref(
  "#ff8c00",
);
const isRouteColorDialogOpen =
  ref(false);

const editingRouteColorId =
  ref<string | null>(null);

const pendingIndividualRouteColor =
  ref("#8fd3ff");
let routeStart: FlowNode | null = null;
let routeEnd: FlowNode | null = null;
let ignoredManualRouteNodeAfterRemove: FlowNode | null = null;
const blockedPipes: SelectionMap = new Map();
const valveBlockedPipeLinks = new Map<string, ValveControlledPipeLink[]>();
const valveControlledPipeLinks = new Map<string, ValveControlledPipeLink[]>();
const manualValveControlledPipes =
  new Map<
    string,
    ManualValveControlledPipe[]
  >();
const highlightedManualValveDraftPipes =
  ref<ManualValveControlledPipe[]>(
    [],
  );
const manualValvePipeDraft =
  ref<ManualValveControlledPipe[]>(
    [],
  );
const isManualValvePipeDefinitionMode =
  ref(false);
const hasManualValvePipeDraftChanges =
  ref(false);
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

const circuitMaterialCache = new Map<string, any>();

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

function getRoutesForNode(
  node: FlowNode,
  options: {
    circuit?: PipeCircuit;
    visibleOnly?: boolean;
    simulationOnly?: boolean;
  } = {},
) {
  const matchingRoutes = savedRoutes.filter((route) => {
    if (options.visibleOnly && route.hidden) {
      return false;
    }

    if (
      options.circuit &&
      route.temperature !== options.circuit
    ) {
      return false;
    }

    if (
      options.simulationOnly &&
      selectedRouteIdsForSimulation.size > 0 &&
      !selectedRouteIdsForSimulation.has(route.id)
    ) {
      return false;
    }

    return routeContainsAdaptedNode(route, node);
  });

  return matchingRoutes.sort(compareRoutesForVisualPriority);
}

function compareRoutesForVisualPriority(
  firstRoute: SavedRoute,
  secondRoute: SavedRoute,
) {
  const nameComparison = firstRoute.name.localeCompare(
    secondRoute.name,
    "pt",
    {
      sensitivity: "base",
      numeric: true,
    },
  );

  if (nameComparison !== 0) {
    return nameComparison;
  }

  return firstRoute.id.localeCompare(secondRoute.id);
}

function getVisualRouteForNode(
  node: FlowNode,
  circuit?: PipeCircuit,
) {
  const routes = getRoutesForNode(node, {
    circuit,
    visibleOnly: true,
    simulationOnly: true,
  });

  return routes[0] ?? null;
}

function getVisibleColorForNode(
  circuit: PipeCircuit,
  node: FlowNode,
) {
  const visibleRoute =
    getVisualRouteForNode(
      node,
      circuit,
    );

  if (visibleRoute) {
    return getSavedRouteGroupColor(
      visibleRoute,
    );
  }

  return getCircuitColorStyle(
    circuit,
  );
}

function shouldColorNode(
  circuit: PipeCircuit,
  node: FlowNode,
) {
    if (
    isValveFocusModeActive.value
  ) {
    return false;
  }

  if (
  isRouteDirectionFocusModeActive.value
) {
  return false;
}

  if (
    selectedSavedRouteIdForEditing.value
  ) {
    const editingRoute =
      savedRoutes.find(
        (route) =>
          route.id ===
          selectedSavedRouteIdForEditing.value,
      );

    return (
      !!editingRoute &&
      !editingRoute.hidden &&
      routeContainsAdaptedNode(
        editingRoute,
        node,
      )
    );
  }

  if (
    selectedRouteIdsForSimulation.size > 0
  ) {
    return getVisualRouteForNode(node, circuit) !== null;
  }

  if (
    isNodeHiddenBySavedRouteVisibility(node)
  ) {
    return false;
  }

  return true;
}

function isManualValveControlledPipeBlocked(
  node: FlowNode,
) {

  const targetPipeKey =
    nodeKey(node);

  for (
    const [
      valveKey,
      controlledPipes,
    ] of manualValveControlledPipes
  ) {
    const controlsTargetPipe =
      controlledPipes.some(
        (pipe) =>
          nodeKey(pipe) ===
          targetPipeKey,
      );

    if (!controlsTargetPipe) {
      continue;
    }

    const valveNode =
      getValveNodeFromDesignationKey(
        valveKey,
      );

    if (!valveNode) {
      continue;
    }

    const valveElement =
      mepElements[
        elementKey(
          valveNode.modelId,
          valveNode.localId,
        )
      ];

    if (
      valveElement?.state ===
      "closed"
    ) {
      return true;
    }
  }

  return false;
}

function shouldCreateArrowForNode(
  circuit: PipeCircuit,
  node: FlowNode,
) {
  if (
    selectedSavedRouteIdForEditing.value
  ) {
    return false;
  }

  if (
    isManualValveControlledPipeBlocked(
      node,
    )
  ) {
    return false;
  }

  return (
    shouldColorNode(
      circuit,
      node,
    ) &&
    shouldIncludeNodeInRouteSimulation(
      circuit,
      node,
    )
  );
}

function getRouteMaterialForNode(
  circuit: PipeCircuit,
  node?: FlowNode,
) {
  if (!node) {
    return getCircuitMaterial(circuit);
  }

  const visibleRoute =
    getVisualRouteForNode(node, circuit);

  if (!visibleRoute) {
    return getCircuitMaterial(circuit);
  }

  const color =
    getSavedRouteGroupColor(
      visibleRoute,
    );

  const materialKey =
    circuit + "|route-color|" + color;

  const cachedMaterial =
    circuitMaterialCache.get(
      materialKey,
    );

  if (cachedMaterial) {
    return cachedMaterial;
  }

  const material =
    new THREE.MeshBasicMaterial({
      color: Number(
        color.replace("#", "0x"),
      ),
      transparent: true,
      opacity: 0.9,
      depthTest: false,
    });

  circuitMaterialCache.set(
    materialKey,
    material,
  );

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
  serializer.classes.abstract.add(...FRAGS.ifcClasses.types);

serializer.relations.set(WEBIFC.IFCRELDEFINESBYTYPE, {
  forRelated: "IsTypedBy",
  forRelating: "Types",
});

serializer.relations.set(WEBIFC.IFCRELASSIGNSTOGROUP, {
  forRelated: "HasAssignments",
  forRelating: "IsGroupedBy",
});

serializer.relations.set(WEBIFC.IFCRELDEFINESBYPROPERTIES, {
  forRelated: "IsDefinedBy",
  forRelating: "DefinesOccurrence",
});

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

  if (
  activeIfcStorageId.value &&
  savedRoutes.length
) {
  await applyAllSavedRoutes();
  await clearBlockedPipes();
} else {
  flowMessage.value =
    `Modelo carregado: ${model.modelId}. Seleciona tubos e atribui os circuitos.`;
}
});

  createBimPanel(components, viewport);
  animateFlow();
});

onBeforeUnmount(() => {
  cancelAnimationFrame(animationFrame);
  clearFlowLayer();
});

function applyIfcPanelLayout() {
  const app = document.getElementById(
    "appGrid",
  ) as BUI.Grid<["main"]> | null;

  if (!app || !bimGridViewport || !bimGridPanel) {
    return;
  }

  if (isIfcPanelCollapsed.value) {
    bimGridPanel.style.display = "none";

    bimGridViewport.style.position = "absolute";
    bimGridViewport.style.inset = "0";
    bimGridViewport.style.width = "100%";
    bimGridViewport.style.height = "100%";

    app.layouts = {
      main: {
        template: `
          "viewport"
          / 1fr
        `,
        elements: {
          viewport: bimGridViewport,
        },
      },
    };
  } else {
    bimGridPanel.style.display = "";

    bimGridViewport.style.position = "";
    bimGridViewport.style.inset = "";
    bimGridViewport.style.width = "";
    bimGridViewport.style.height = "";

    app.layouts = {
      main: {
        template: `
          "panel viewport"
          / 23rem 1fr
        `,
        elements: {
          panel: bimGridPanel,
          viewport: bimGridViewport,
        },
      },
    };
  }

  app.layout = "main";

  setTimeout(() => {
    void fragmentManager?.core.update(true);
  }, 0);
}

function toggleIfcPanelCollapsed() {
  isIfcPanelCollapsed.value = !isIfcPanelCollapsed.value;
  applyIfcPanelLayout();
}

async function restoreSelectedCircuitColors(
  _previousSelection: SelectionMap,
) {
  if (
    isValveFocusModeActive.value ||
    isRouteDirectionFocusModeActive.value ||
    isApplyingSavedRouteHighlight.value ||
    highlightedSavedRouteId.value
  ) {
    return;
  }

  await clearPersistentCircuitHighlights();

  const idsByCircuitAndModel =
    new Map<
      string,
      {
        circuit: PipeCircuit;
        modelId: string;
        localIds: number[];
      }
    >();

  for (
    const circuit of
      getAllKnownCircuitKeys()
  ) {
    const assignmentMap =
      manualAssignments[circuit];

    if (!assignmentMap) {
      continue;
    }

    if (
      !shouldShowCircuitInSimulation(
        circuit,
      )
    ) {
      continue;
    }

    for (
      const [modelId, localIds] of
        assignmentMap
    ) {
      if (!localIds.size) {
        continue;
      }

      idsByCircuitAndModel.set(
        circuit + "|" + modelId,
        {
          circuit,
          modelId,
          localIds: [
            ...localIds,
          ],
        },
      );
    }
  }

  for (
    const {
      circuit,
      modelId,
      localIds,
    } of idsByCircuitAndModel.values()
  ) {
    const model =
      loadedModels.get(modelId);

    if (
      !model ||
      !localIds.length
    ) {
      continue;
    }

    const idsByColor =
      new Map<string, number[]>();

    for (const localId of localIds) {
      const node: FlowNode = {
        modelId,
        localId,
      };

      if (
        !shouldColorNode(
          circuit,
          node,
        )
      ) {
        continue;
      }

      const color =
  getVisibleColorForNode(
    circuit,
    node,
  );

      const colorIds =
        idsByColor.get(color) ?? [];

      colorIds.push(localId);

      idsByColor.set(
        color,
        colorIds,
      );
    }

    for (
      const [color, colorLocalIds] of
        idsByColor
    ) {
      const persistentStyleName =
  "persistent-circuit-" +
  circuit +
  "-" +
  modelId +
  "-" +
  color.replace(
    "#",
    "",
  );

      await applyPersistentCircuitHighlight(
        persistentStyleName,
        color,
        modelId,
        colorLocalIds,
      );
    }
  }

  await fragmentManager.core.update(
    true,
  );
}

function resetManualChangesOnly() {
  const confirmed =
    window.confirm(
      "Queres repor apenas as alterações manuais?\n\n" +
        "Serão repostos:\n" +
        "• sentidos definidos, sincronizados ou invertidos;\n" +
        "• proteção, união e agrupamento de caminhos;\n" +
        "• cores personalizadas;\n" +
        "• caminhos ocultados e setas ocultadas;\n" +
        "• nomes e estados manuais das válvulas;\n" +
        "• associações entre válvulas e caminhos.\n\n" +
        "Os caminhos guardados, ciclos e circuitos serão mantidos.\n\n" +
        "Esta ação não pode ser anulada.",
    );

  if (!confirmed) {
    flowMessage.value =
      "Reposição das alterações manuais cancelada.";

    return;
  }

  const savedRoutesText =
    getActiveIfcStorageItem(
      ROUTES_STORAGE_KEY,
    );

  if (savedRoutesText) {
    try {
      const storedRoutes =
        JSON.parse(
          savedRoutesText,
        ) as SavedRoute[];

      const resetRoutes =
        storedRoutes.map(
          (route) => {
            const resetRoute = {
              ...route,

              hidden:
                false,

              locked:
                false,
            };

            delete resetRoute.groupId;

            delete resetRoute.originalCircuitColor;

            delete resetRoute.customColor;

            delete resetRoute.directionStarts;

            delete resetRoute.directionEnds;

            delete resetRoute.needsDirectionRedefinition;

            delete resetRoute.protectedDirections;

            delete resetRoute.mergeBackup;

            return resetRoute;
          },
        );

      setActiveIfcStorageItem(
        ROUTES_STORAGE_KEY,
        JSON.stringify(
          resetRoutes,
        ),
      );
    } catch (error) {
      console.error(
        "Erro ao repor os caminhos guardados:",
        error,
      );
    }
  }

  const savedMepElementsText =
    getActiveIfcStorageItem(
      MEP_ELEMENTS_STORAGE_KEY,
    );

  if (savedMepElementsText) {
    try {
      const storedElements =
        JSON.parse(
          savedMepElementsText,
        ) as Record<
          string,
          MepElement
        >;

      for (
        const element of
          Object.values(
            storedElements,
          )
      ) {
        delete element.name;

        if (
  isValveElementType(
    element.elementType,
  )
) {
  element.state =
    getNormalValveState(
      element.elementType,
    );
}

      }

      setActiveIfcStorageItem(
        MEP_ELEMENTS_STORAGE_KEY,
        JSON.stringify(
          storedElements,
        ),
      );
    } catch (error) {
      console.error(
        "Erro ao repor as alterações dos elementos:",
        error,
      );
    }
  }

  const manualChangeStorageKeys = [
    MANUAL_VALVE_CONTROLLED_PIPES_STORAGE_KEY,
    ROUTE_GROUPS_STORAGE_KEY,
    REVERSED_DIRECTIONS_STORAGE_KEY,
    SYNCED_PIPE_DIRECTIONS_STORAGE_KEY,
    HIDDEN_FLOW_ARROWS_STORAGE_KEY,
    VALVE_PIPE_LINKS_STORAGE_KEY,
    EXCLUDED_VALVES_STORAGE_KEY,
  ];

  for (
    const storageKey of
      manualChangeStorageKeys
  ) {
    removeActiveIfcStorageItem(
      storageKey,
    );
  }

  window.alert(
    "As alterações manuais foram repostas.\n\n" +
      "Os caminhos, ciclos e circuitos foram mantidos.\n\n" +
      "A aplicação será reiniciada.",
  );

  window.location.reload();
}

function resetAllManualConfiguration() {
  const confirmed =
    window.confirm(
      "ATENÇÃO: esta ação vai apagar tudo o que está guardado para o IFC atual.\n\n" +
        "Serão apagados:\n" +
        "• caminhos guardados;\n" +
        "• ciclos e circuitos;\n" +
        "• nomes e cores;\n" +
        "• sentidos e correções;\n" +
        "• grupos, uniões e proteções;\n" +
        "• classificações dos elementos;\n" +
        "• válvulas, estados e associações;\n" +
        "• tubos controlados pelas válvulas;\n" +
        "• exclusões e configurações de visualização.\n\n" +
        "O ficheiro IFC original não será apagado.\n\n" +
        "Esta ação não pode ser anulada.\n\n" +
        "Queres continuar?",
    );

  if (!confirmed) {
    flowMessage.value =
      "Apagar tudo foi cancelado.";

    return;
  }

  const securityText =
    window.prompt(
      "Confirmação de segurança\n\n" +
        "Para apagar todos os dados deste IFC, escreve exatamente:\n\n" +
        "APAGAR TUDO",
      "",
    );

  if (securityText === null) {
    flowMessage.value =
      "Apagar tudo foi cancelado.";

    return;
  }

  if (
    securityText.trim() !==
    "APAGAR TUDO"
  ) {
    window.alert(
      "O texto de segurança está incorreto.\n\n" +
        "Nenhum dado foi apagado.",
    );

    flowMessage.value =
      "O texto de segurança estava incorreto. Nenhum dado foi apagado.";

    return;
  }

  const allConfigurationStorageKeys = [
    MEP_ELEMENTS_STORAGE_KEY,
    ROUTES_STORAGE_KEY,
    ROUTE_GROUPS_STORAGE_KEY,
    WATER_CYCLE_COUNT_STORAGE_KEY,
    WATER_CYCLE_NAMES_STORAGE_KEY,
    CYCLE_CIRCUITS_STORAGE_KEY,
    REVERSED_DIRECTIONS_STORAGE_KEY,
    HIDDEN_FLOW_ARROWS_STORAGE_KEY,
    PIPE_TYPE_FLOW_NODES_STORAGE_KEY,
    SYNCED_PIPE_DIRECTIONS_STORAGE_KEY,
    VALVE_PIPE_LINKS_STORAGE_KEY,
    EXCLUDED_VALVES_STORAGE_KEY,
    MANUAL_VALVE_CONTROLLED_PIPES_STORAGE_KEY,
  ];

  for (
    const storageKey of
      allConfigurationStorageKeys
  ) {
    removeActiveIfcStorageItem(
      storageKey,
    );
  }

  window.alert(
    "Todos os dados guardados para este IFC foram apagados.\n\n" +
      "A aplicação será reiniciada para começares de novo.",
  );

  window.location.reload();
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

  modelHighlighter = highlighter;

  highlighter.setup({ world });
  highlighter.styles.set(
  "saved-route-highlight",
  {
    color: new THREE.Color(0x00e5ff),
    opacity: 0.95,
    transparent: true,
    renderedFaces:
      FRAGS.RenderedFaces.TWO,
  },
);
  highlighter.styles.set("select", {
  color: new THREE.Color(0x00ff66),
  opacity: 0.85,
  transparent: true,
  renderedFaces: FRAGS.RenderedFaces.TWO,
});

  highlighter.events.select.onHighlight.add(async (modelIdMap) => {
  replaceSelection(modelIdMap);

  selectedIfcDetailsText.value = "";
isIfcDetailsPanelOpen.value = false;

updatePropertiesTable({ modelIdMap });

await showSelectedMepElementInfo();

updateSelectedTubeRouteInfo();

addSelectedNodeToManualRoute();

if (isManualRouteRecording.value && manualRouteNodes.length > 0) {
  await updateManualRoutePreviewHighlight();
  return;
}

const selectedValveNode =
  getFirstSelectedValveNode();

if (selectedValveNode) {
  await syncValveDesignationPanelFromNode(
    selectedValveNode,
  );
}

await syncSelectedValveAssociationRoute();
});

highlighter.events.select.onClear.add(
  async () => {
    const previousSelection:
      SelectionMap = new Map();

    for (
      const [modelId, localIds] of
        selectedItems
    ) {
      previousSelection.set(
        modelId,
        new Set(localIds),
      );
    }

    selectedItems.clear();
    selectedCount.value = 0;
    selectedIfcDetailsText.value = "";
    isIfcDetailsPanelOpen.value = false;

    selectedMepElementInfo.value =
      "Nenhum elemento classificado selecionado.";

    selectedTubeRouteInfo.value = "";

    updatePropertiesTable({
      modelIdMap: {},
    });

    if (isManualRouteRecording.value) {
      return;
    }

    if (highlightedSavedRouteId.value) {
      await fragmentManager.core.update(true);
      return;
    }

    if (
  !isApplyingSavedRouteHighlight.value &&
  !isValveFocusModeActive.value
) {
  await restoreSelectedCircuitColors(
    previousSelection,
  );
}
  },
);

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

  clearActiveModelConfigurationFromMemory();

activeIfcStorageId.value =
  createIfcStorageId(file);

loadActiveIfcConfiguration();

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

function isSimulationCycleSelected(
  cycleNumber: number,
) {
  return selectedSimulationCycles.has(
    cycleNumber,
  );
}

function toggleSimulationCycleSelection(
  cycleNumber: number,
) {
  if (
    selectedSimulationCycles.has(
      cycleNumber,
    )
  ) {
    selectedSimulationCycles.delete(
      cycleNumber,
    );

    return;
  }

  selectedSimulationCycles.add(
    cycleNumber,
  );
}

function selectAllSimulationCycles() {
  selectedSimulationCycles.clear();

  for (
    let cycleNumber = 1;
    cycleNumber <= waterCycleCount.value;
    cycleNumber++
  ) {
    selectedSimulationCycles.add(
      cycleNumber,
    );
  }
}

function clearSimulationCycleSelection() {
  selectedSimulationCycles.clear();
}

function getSelectedSimulationCyclesLabel() {
  return [...selectedSimulationCycles]
    .sort(
      (firstCycle, secondCycle) =>
        firstCycle - secondCycle,
    )
    .map((cycleNumber) =>
      getCycleDisplayName(cycleNumber),
    )
    .join(", ");
}

function shouldShowCircuitInSimulation(
  circuit: PipeCircuit,
) {
  if (!isCycleViewFilterActive.value) {
    return true;
  }

  const circuitCycleNumber =
    getCircuitCycleNumber(circuit);

  return selectedSimulationCycles.has(
    circuitCycleNumber,
  );
}

async function resetAllCircuitHighlights() {
  const loadedModelIds = [
    ...loadedModels.keys(),
  ];

  if (!loadedModelIds.length) {
    return;
  }

  const fallbackModelId =
    loadedModelIds[0];

  const idsByModel =
    new Map<string, Set<number>>();

  function addNode(
    modelId: string,
    localId: number,
  ) {
    const validModelId =
      loadedModels.has(modelId)
        ? modelId
        : fallbackModelId;

    const modelIds =
      idsByModel.get(validModelId) ??
      new Set<number>();

    modelIds.add(localId);

    idsByModel.set(
      validModelId,
      modelIds,
    );
  }

  // Elementos atribuídos diretamente aos circuitos.
  for (
    const circuit of
      getAllKnownCircuitKeys()
  ) {
    const assignmentMap =
      manualAssignments[circuit];

    if (!assignmentMap) {
      continue;
    }

    for (
      const [modelId, localIds] of
        assignmentMap
    ) {
      for (const localId of localIds) {
        addNode(modelId, localId);
      }
    }
  }

  // Elementos existentes nos caminhos guardados.
  for (const route of savedRoutes) {
    for (const node of route.path) {
      addNode(
        node.modelId,
        node.localId,
      );
    }
  }

  // Remover as cores de todos os ciclos.
  for (
    const [modelId, localIds] of
      idsByModel
  ) {
    const model =
      loadedModels.get(modelId);

    if (!model || !localIds.size) {
      continue;
    }

    await model.resetHighlight(
      [...localIds],
    );
  }

  await fragmentManager.core.update(true);

  // Esperar a limpeza ser renderizada antes
  // de aplicar as cores do ciclo selecionado.
  await new Promise<void>((resolve) => {
    requestAnimationFrame(() => {
      resolve();
    });
  });
}

function cloneProtectedRouteNode(
  node: FlowNode,
): FlowNode {
  return {
    modelId:
      node.modelId,

    localId:
      node.localId,
  };
}

function cloneProtectedRouteDirection(
  direction:
    ProtectedRouteDirection,
): ProtectedRouteDirection {
  return {
    node:
      cloneProtectedRouteNode(
        direction.node,
      ),

    previous:
      direction.previous
        ? cloneProtectedRouteNode(
            direction.previous,
          )
        : null,

    next:
      direction.next
        ? cloneProtectedRouteNode(
            direction.next,
          )
        : null,

    reversed:
      direction.reversed,
  };
}

function createProtectedRouteSnapshot(
  route: SavedRoute,
): ProtectedRouteSnapshot {
  return {
    name:
      route.name,

    temperature:
      route.temperature,

    path:
      route.path.map(
        cloneProtectedRouteNode,
      ),

    hidden:
      !!route.hidden,

    groupId:
      route.groupId ??
      null,

    groupName:
      getSavedRouteGroupName(
        route,
      ),

    effectiveColor:
      getSavedRouteGroupColor(
        route,
      ),

    circuitLabel:
      getRouteCircuitDisplayLabel(
        route,
      ),

    originalCircuitColor:
      route.originalCircuitColor ??
      null,

    customColor:
      route.customColor ??
      null,

    directionStarts:
      (
        route.directionStarts ??
        []
      ).map(
        cloneProtectedRouteNode,
      ),

    directionEnds:
      (
        route.directionEnds ??
        []
      ).map(
        cloneProtectedRouteNode,
      ),

    needsDirectionRedefinition:
      !!route
        .needsDirectionRedefinition,

    protectedDirections:
      (
        route.protectedDirections ??
        []
      ).map(
        cloneProtectedRouteDirection,
      ),
  };
}

function restoreProtectedRouteSnapshot(
  route: SavedRoute,
) {
  if (
    !route.locked ||
    !route.protectedSnapshot
  ) {
    return;
  }

  const snapshot =
    route.protectedSnapshot;

  route.name =
    snapshot.name;

  route.temperature =
    snapshot.temperature;

  route.path =
    snapshot.path.map(
      cloneProtectedRouteNode,
    );

  route.hidden =
    snapshot.hidden;

  if (snapshot.groupId) {
    route.groupId =
      snapshot.groupId;
  } else {
    delete route.groupId;
  }

  if (
    snapshot.originalCircuitColor
  ) {
    route.originalCircuitColor =
      snapshot.originalCircuitColor;
  } else {
    delete route
      .originalCircuitColor;
  }

  if (snapshot.customColor) {
    route.customColor =
      snapshot.customColor;
  } else {
    delete route.customColor;
  }

  route.directionStarts =
    snapshot.directionStarts.map(
      cloneProtectedRouteNode,
    );

  route.directionEnds =
    snapshot.directionEnds.map(
      cloneProtectedRouteNode,
    );

  route.needsDirectionRedefinition =
    snapshot
      .needsDirectionRedefinition;

  route.protectedDirections =
    snapshot
      .protectedDirections
      .map(
        cloneProtectedRouteDirection,
      );
}

function restoreAllProtectedRouteSnapshots() {
  for (
    const route of savedRoutes
  ) {
    restoreProtectedRouteSnapshot(
      route,
    );
  }
}

function isNodeInProtectedRoute(
  node: FlowNode,
) {
  return savedRoutes.some(
    (route) =>
      route.locked &&
      route.path.some(
        (routeNode) =>
          isSameNode(
            routeNode,
            node,
          ),
      ),
  );
}

function applyProtectedRouteDirections(
  route: SavedRoute,
) {
  const protectedDirections =
    route.protectedDirections ?? [];

  for (
    const direction of
      protectedDirections
  ) {
    const directionKey =
      automaticDirectionKey(
        route.temperature,
        direction.node,
      );

    if (
      direction.previous ||
      direction.next
    ) {
      automaticDirectionNeighbors.set(
        directionKey,
        {
          previous:
            direction.previous
              ? {
                  modelId:
                    direction.previous
                      .modelId,

                  localId:
                    direction.previous
                      .localId,
                }
              : null,

          next:
            direction.next
              ? {
                  modelId:
                    direction.next
                      .modelId,

                  localId:
                    direction.next
                      .localId,
                }
              : null,
        },
      );
    }

    let reversedIds =
      reversedPipeDirections.get(
        direction.node.modelId,
      );

    if (direction.reversed) {
      if (!reversedIds) {
        reversedIds =
          new Set<number>();

        reversedPipeDirections.set(
          direction.node.modelId,
          reversedIds,
        );
      }

      reversedIds.add(
        direction.node.localId,
      );

      continue;
    }

    reversedIds?.delete(
      direction.node.localId,
    );

    if (
      reversedIds &&
      reversedIds.size === 0
    ) {
      reversedPipeDirections.delete(
        direction.node.modelId,
      );
    }
  }
}

async function reapplyAllSavedRouteDirections() {
  restoreAllProtectedRouteSnapshots();

  const editableRoutes =
    savedRoutes.filter(
      (route) =>
        !route.locked,
    );

  const protectedRoutes =
    savedRoutes.filter(
      (route) =>
        route.locked,
    );

  for (
    const route of
      editableRoutes
  ) {
    const starts =
      route.directionStarts ??
      [];

    const ends =
      route.directionEnds ??
      [];

    if (
      !starts.length ||
      !ends.length
    ) {
      continue;
    }

    const directionPaths =
      await findSavedRouteDirectionPaths(
        route,
        starts,
        ends,
      );

    if (
      !directionPaths.length
    ) {
      continue;
    }

    const {
      compatiblePaths,
    } =
      selectCompatibleDirectionPaths(
        directionPaths,
      );

    if (
      !compatiblePaths.length
    ) {
      continue;
    }

    applySavedRouteDirectionPaths(
      route,
      compatiblePaths,
    );
  }

  for (
    const route of
      protectedRoutes
  ) {
    restoreProtectedRouteSnapshot(
      route,
    );

    applyProtectedRouteDirections(
      route,
    );
  }

  saveReversedDirectionsToStorage();
}

async function rebuildManualFlowLayer(
  shouldReapplySavedDirections = true,
) {
  restoreAllProtectedRouteSnapshots();

  const currentRunId =
    ++flowPreparationRunId;

  const isCurrentRun = () =>
    currentRunId ===
    flowPreparationRunId;

  const setCurrentProgress = (
    progress: number,
  ) => {
    if (!isCurrentRun()) {
      return;
    }

    flowPreparationProgress.value =
      progress;
  };

  isPreparingFlowAnimation.value = true;
  isFlowAnimationReady.value = false;
  hasFlowPreparationError.value = false;

  setCurrentProgress(5);

  try {
    clearFlowVisuals(true);

    const hasAssignments =
      countAssignments() > 0;

    const hasConnections =
      flowConnections.length > 0;

    if (
      !hasAssignments &&
      !hasConnections
    ) {
      setCurrentProgress(0);
      hasFlowPreparationError.value = true;

      flowMessage.value =
        "Ainda não há tubos marcados.";

      return;
    }

        setCurrentProgress(10);

        if (shouldReapplySavedDirections) {
  await reapplyAllSavedRouteDirections();
}

    if (!isCurrentRun()) {
      return;
    }

    await clearPersistentCircuitHighlights();

    if (!isCurrentRun()) {
      return;
    }

    await resetAllCircuitHighlights();

if (!isCurrentRun()) {
  return;
}

setCurrentProgress(20);

    const visibleCircuits =
      getAllKnownCircuitKeys().filter(
        (circuit) =>
          shouldShowCircuitInSimulation(
            circuit,
          ),
      );

    const circuitCount =
      visibleCircuits.length;

    if (!circuitCount) {
      setCurrentProgress(0);
      hasFlowPreparationError.value = true;

      flowMessage.value =
        "Não existem circuitos visíveis para preparar.";

      return;
    }

    for (
      let circuitIndex = 0;
      circuitIndex < circuitCount;
      circuitIndex++
    ) {
      const circuit =
        visibleCircuits[circuitIndex];

      await addAssignmentsToScene(
  circuit,
);

if (!isCurrentRun()) {
  return;
}

const completedCircuits =
  circuitIndex + 1;

      setCurrentProgress(
  Math.round(
    20 +
    (
      completedCircuits /
      circuitCount
    ) *
    70,
  ),
);
    }

    updateManualStats();

    setCurrentProgress(95);

    const shouldKeepAnimating =
      !isFlowManuallyPaused.value &&
      (
        isManualFlowAnimationRunning.value ||
        isCentralSimulationRunning.value
      );

    isFlowing.value =
      shouldKeepAnimating &&
      pipeParticles.length > 0;

    if (highlightedSavedRouteId.value) {
      await enforceActiveSavedRouteHighlight();

      isFlowing.value = false;

      isCentralSimulationRunning.value =
        false;

      isManualFlowAnimationRunning.value =
        false;

      isFlowManuallyPaused.value = true;

      if (isCurrentRun()) {
  setCurrentProgress(100);

  isFlowAnimationReady.value = true;
  hasFlowPreparationError.value = false;
}

      return;
    }

    await fragmentManager.core.update(
  true,
);

if (!isCurrentRun()) {
  return;
}

setCurrentProgress(100);

isFlowAnimationReady.value = true;
hasFlowPreparationError.value = false;
  } catch (error) {
  console.error(
    "Erro ao preparar a animação:",
    error,
  );

  if (isCurrentRun()) {
    hasFlowPreparationError.value = true;
    isFlowAnimationReady.value = false;

    flowMessage.value =
      "Não foi possível preparar completamente a animação.";
  }
  } finally {
  if (isCurrentRun()) {
    isPreparingFlowAnimation.value =
      false;
  }
}
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
  (localId) => {
    const node: FlowNode = {
      modelId,
      localId,
    };

    return (
      !isNodeHiddenBySavedRouteVisibility(
        node,
      ) &&
      shouldCreateArrowForNode(
        temperature,
        node,
      )
    );
  },
);

if (hiddenIds.length) {
  await model.resetHighlight(hiddenIds);
}

if (!ids.length) continue;

    const idsByColor =
  new Map<string, number[]>();

for (const localId of ids) {
  const node: FlowNode = {
    modelId,
    localId,
  };

  const color =
  getVisibleColorForNode(
    temperature,
    node,
  );

  const colorIds =
    idsByColor.get(color) ?? [];

  colorIds.push(localId);

  idsByColor.set(
    color,
    colorIds,
  );
}

for (
  const [color, colorLocalIds] of
    idsByColor
) {
  const persistentStyleName =
  "persistent-circuit-" +
  temperature +
  "-" +
  modelId +
  "-" +
  color.replace(
    "#",
    "",
  );

  await applyPersistentCircuitHighlight(
    persistentStyleName,
    color,
    modelId,
    colorLocalIds,
  );
}

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

const node = {
  modelId,
  localId,
};

if (
  !shouldCreateArrowForNode(
    temperature,
    node,
  )
) {
  continue;
}

if (
  !pipeTypeFlowNodes.has(
    nodeKey(node),
  )
) {
  continue;
}

const hints =
  await getPipeDirectionHints(node);

const hasAutomaticDirection =
  !!hints.upstream ||
  !!hints.downstream;

const belongsToAutomaticPath =
  automaticDirectionNeighbors.has(
    automaticDirectionKey(
      temperature,
      node,
    ),
  );

if (
  belongsToAutomaticPath &&
  !hasAutomaticDirection
) {
  continue;
}

const geometryAxis =
  await getPipeGeometryAxis(
    node,
  );

addPipeParticles(
  box,
  temperature,
  hints,
  node,
  geometryAxis,
);
    }
  }
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

function saveAutomaticRoutes() {
  if (!automaticOrderedCircuitNodes.size) {
    flowMessage.value =
      "Cria primeiro os caminhos automáticos.";

    return;
  }

  let savedCount = 0;
  let existingCount = 0;

  for (
    const [circuitKey, orderedNodes] of
      automaticOrderedCircuitNodes
  ) {
    if (orderedNodes.length < 2) {
      continue;
    }

    const circuit =
      getCycleCircuitDefinition(
        circuitKey,
      );

    if (!circuit) {
      continue;
    }

    const automaticNodeKeys =
      new Set(
        orderedNodes.map(
          (node) =>
            nodeKey({
              modelId: node.modelId,
              localId: node.localId,
            }),
        ),
      );

    const existingRoute =
      savedRoutes.find(
        (route) => {
          if (
            route.temperature !==
              circuitKey ||
            route.path.length !==
              orderedNodes.length
          ) {
            return false;
          }

          return route.path.every(
            (node) =>
              automaticNodeKeys.has(
                nodeKey(node),
              ),
          );
        },
      );

    if (existingRoute) {
      existingCount++;

      continue;
    }

    const routeNumber =
      getNextRouteNumberForCircuit(
        circuitKey,
      );

    savedRoutes.push({
      id: crypto.randomUUID(),

      name:
        circuit.name +
        " - Caminho " +
        routeNumber,

      temperature: circuitKey,

      path: orderedNodes.map(
        (node) => ({
          modelId: node.modelId,
          localId: node.localId,
        }),
      ),

      hidden: false,
      locked: false,
    });

    savedCount++;
  }

  saveRoutesToStorage();

  flowMessage.value =
    savedCount +
    " caminho(s) automático(s) guardado(s). " +
    existingCount +
    " caminho(s) existente(s) mantido(s) com a respetiva configuração.";
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

async function undoSavedRouteMerge(
  routeId: string,
) {
  const mergedRouteIndex =
    savedRoutes.findIndex(
      (route) =>
        route.id === routeId,
    );

  if (mergedRouteIndex === -1) {
    flowMessage.value =
      "Não foi possível encontrar o percurso unido.";

    return;
  }

  const mergedRoute =
    savedRoutes[mergedRouteIndex];

  const mergeBackup =
    mergedRoute.mergeBackup;

  if (!mergeBackup) {
    flowMessage.value =
      "Este percurso não foi criado através de uma união.";

    return;
  }

  if (mergedRoute.locked) {
    flowMessage.value =
      'O percurso "' +
      mergedRoute.name +
      '" está protegido. Desprotege primeiro.';

    return;
  }

  const shouldUndo =
    window.confirm(
      "Tens a certeza de que queres desfazer esta união?\n\n" +
      'O percurso unido "' +
      mergedRoute.name +
      '" será eliminado e serão recuperados ' +
      mergeBackup.sourceRoutes.length +
      " percursos anteriores.",
    );

  if (!shouldUndo) {
    flowMessage.value =
      "A reposição dos percursos anteriores foi cancelada.";

    return;
  }

  if (
    highlightedSavedRouteId.value ===
    mergedRoute.id
  ) {
    await modelHighlighter?.clear(
      "saved-route-highlight",
    );

    highlightedSavedRouteId.value =
      null;
  }

  selectedRouteIdsForSimulation.delete(
    mergedRoute.id,
  );

  selectedRouteIdsForGrouping.delete(
    mergedRoute.id,
  );

  savedRoutes.splice(
    mergedRouteIndex,
    1,
  );

  for (
    const sourceRoute of
      mergeBackup.sourceRoutes
  ) {
    const restoredRoute: SavedRoute = {
      id: sourceRoute.id,
      name: sourceRoute.name,

      temperature:
        sourceRoute.temperature,

      path: sourceRoute.path.map(
        (node) => ({
          modelId: node.modelId,
          localId: node.localId,
        }),
      ),

      hidden: sourceRoute.hidden,
      locked: sourceRoute.locked,
      groupId: sourceRoute.groupId,

      originalCircuitColor:
        sourceRoute.originalCircuitColor,

      customColor:
        sourceRoute.customColor,

      directionStarts:
        sourceRoute.directionStarts?.map(
          (node) => ({
            modelId: node.modelId,
            localId: node.localId,
          }),
        ),

      directionEnds:
        sourceRoute.directionEnds?.map(
          (node) => ({
            modelId: node.modelId,
            localId: node.localId,
          }),
        ),

        mergeBackup:
  sourceRoute.mergeBackup
    ? JSON.parse(
        JSON.stringify(
          sourceRoute.mergeBackup,
        ),
      )
    : undefined,
    };

    savedRoutes.push(
      restoredRoute,
    );
  }

  for (
    const sourceRoute of
      mergeBackup.sourceRoutes
  ) {
    for (const node of sourceRoute.path) {
      for (
        const circuit of
          getAllKnownCircuitKeys()
      ) {
        getAssignmentSet(
          circuit,
          node.modelId,
        ).delete(
          node.localId,
        );
      }

      getAssignmentSet(
        sourceRoute.temperature,
        node.modelId,
      ).add(
        node.localId,
      );
    }
  }

  for (
    const sourceValveLink of
      mergeBackup.sourceValveLinks
  ) {
    valveControlledPipeLinks.set(
      sourceValveLink.valveKey,

      sourceValveLink.linkedPipes.map(
        (linkedPipe) => ({
          ...linkedPipe,
        }),
      ),
    );
  }

  for (
    const sourceBlockedValveLink of
      mergeBackup.sourceBlockedValveLinks ??
      []
  ) {
    valveBlockedPipeLinks.set(
      sourceBlockedValveLink.valveKey,

      sourceBlockedValveLink.linkedPipes.map(
        (linkedPipe) => ({
          ...linkedPipe,
        }),
      ),
    );
  }

  for (
    const sourceRouteId of
      mergeBackup.sourceSimulationRouteIds
  ) {
    selectedRouteIdsForSimulation.add(
      sourceRouteId,
    );
  }

  circuitMaterialCache.clear();

  saveRoutesToStorage();
  saveValvePipeLinksToStorage();

  updateManualStats();
  updateBlockedCount();

  flowConnections.splice(0);

  if (
    countAssignments() > 0 ||
    savedRoutes.length > 0
  ) {
    await applyAllSavedRoutes();
  } else {
    clearFlowVisuals();

    await fragmentManager.core.update(
      true,
    );
  }

  flowMessage.value =
    'A união "' +
    mergedRoute.name +
    '" foi desfeita. Foram recuperados ' +
    mergeBackup.sourceRoutes.length +
    " percursos anteriores.";
}

async function finishEditedRouteChange() {
  selectedSavedRouteIdForEditing.value =
    "";

  highlightedSavedRouteId.value =
    null;

  selectedItems.clear();

  selectedCount.value =
    0;

  selectedTubeRouteInfo.value =
    "";

  selectedIfcDetailsText.value =
    "";

  isIfcDetailsPanelOpen.value =
    false;

  if (modelHighlighter) {
    await modelHighlighter.clear(
      "saved-route-highlight",
    );

    await modelHighlighter.clear(
      "select",
    );
  }

  isFlowing.value =
    false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value =
    true;

  await fragmentManager.core.update(
    true,
  );
}

async function addSelectedPipesToEditedRoute() {
  const route =
    savedRoutes.find(
      (savedRoute) =>
        savedRoute.id ===
        selectedSavedRouteIdForEditing.value,
    );

  if (!route) {
    flowMessage.value =
      "Seleciona primeiro o caminho que queres editar.";

    return;
  }

  if (route.locked) {
    flowMessage.value =
      'O caminho "' +
      route.name +
      '" está protegido. Desprotege primeiro.';

    return;
  }

  if (!selectedCount.value) {
    flowMessage.value =
      "Seleciona primeiro um ou mais tubos no modelo.";

    return;
  }

  const nodesToAdd: FlowNode[] = [];

  const nodesToAddKeys =
    new Set<string>();

  for (
    const [modelId, localIds] of
      selectedItems
  ) {
    for (const localId of localIds) {
      const selectedNode: FlowNode = {
        modelId,
        localId,
      };

      if (
        routeContainsAdaptedNode(
          route,
          selectedNode,
        )
      ) {
        continue;
      }

      const selectedKey =
        nodeKey(
          selectedNode,
        );

      if (
        nodesToAddKeys.has(
          selectedKey,
        )
      ) {
        continue;
      }

      nodesToAddKeys.add(
        selectedKey,
      );

      nodesToAdd.push({
        modelId,
        localId,
      });
    }
  }

  if (!nodesToAdd.length) {
    flowMessage.value =
      "Todos os tubos selecionados já pertencem ao caminho.";

    return;
  }

  const shouldAdd =
    window.confirm(
      "Queres adicionar " +
        nodesToAdd.length +
        ' tubo(s) ao caminho "' +
        route.name +
        '"?\n\n' +
        "O sentido atual será apagado e terás de o definir novamente.",
    );

  if (!shouldAdd) {
    flowMessage.value =
      "Adição dos tubos cancelada.";

    return;
  }

  route.path.push(
    ...nodesToAdd.map(
      (node) => ({
        modelId:
          node.modelId,

        localId:
          node.localId,
      }),
    ),
  );

  delete route.directionStarts;
  delete route.directionEnds;

  route.needsDirectionRedefinition =
    true;

  route.locked =
    false;

  selectedRouteIdsForSimulation.delete(
    route.id,
  );

  for (
    const routeNode of
      route.path
  ) {
    const reversedIds =
      reversedPipeDirections.get(
        routeNode.modelId,
      );

    reversedIds?.delete(
      routeNode.localId,
    );

    if (
      reversedIds &&
      reversedIds.size === 0
    ) {
      reversedPipeDirections.delete(
        routeNode.modelId,
      );
    }

    const syncedIds =
      syncedPipeDirections.get(
        routeNode.modelId,
      );

    syncedIds?.delete(
      routeNode.localId,
    );

    if (
      syncedIds &&
      syncedIds.size === 0
    ) {
      syncedPipeDirections.delete(
        routeNode.modelId,
      );
    }
  }

  for (
    const addedNode of
      nodesToAdd
  ) {
    for (
      const circuitKey of
        getAllKnownCircuitKeys()
    ) {
      getAssignmentSet(
        circuitKey,
        addedNode.modelId,
      ).delete(
        addedNode.localId,
      );
    }

    getAssignmentSet(
      route.temperature,
      addedNode.modelId,
    ).add(
      addedNode.localId,
    );
  }

  saveRoutesToStorage();
  saveReversedDirectionsToStorage();
  saveSyncedPipeDirectionsToStorage();

  updateManualStats();

  isFlowing.value =
    false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value =
    true;

  await applyAllSavedRoutes();

await finishEditedRouteChange();

flowMessage.value =
    nodesToAdd.length +
    ' tubo(s) adicionado(s) ao caminho "' +
    route.name +
    '". Define novamente o sentido antes de proteger ou simular.';
}


async function removeSelectedPipesFromEditedRoute() {
  const route =
    savedRoutes.find(
      (savedRoute) =>
        savedRoute.id ===
        selectedSavedRouteIdForEditing.value,
    );

  if (!route) {
    flowMessage.value =
      "Seleciona primeiro o caminho que queres editar.";

    return;
  }

  if (route.locked) {
    flowMessage.value =
      'O caminho "' +
      route.name +
      '" está protegido. Desprotege primeiro.';

    return;
  }

  if (!selectedCount.value) {
    flowMessage.value =
      "Seleciona primeiro um ou mais tubos no modelo.";

    return;
  }

  const selectedNodeKeys =
    new Set<string>();

  for (
    const [modelId, localIds] of
      selectedItems
  ) {
    for (const localId of localIds) {
      selectedNodeKeys.add(
        elementKey(
          modelId,
          localId,
        ),
      );
    }
  }

  const adaptedRouteNodes =
    getAdaptedSavedRouteNodes(
      route,
    );

  const indexesToRemove =
    new Set<number>();

  for (
    let nodeIndex = 0;
    nodeIndex < adaptedRouteNodes.length;
    nodeIndex++
  ) {
    const node =
      adaptedRouteNodes[nodeIndex];

    if (
      selectedNodeKeys.has(
        elementKey(
          node.modelId,
          node.localId,
        ),
      )
    ) {
      indexesToRemove.add(
        nodeIndex,
      );
    }
  }

  if (!indexesToRemove.size) {
    flowMessage.value =
      "Nenhum dos tubos selecionados pertence ao caminho escolhido.";

    return;
  }

  if (
    route.path.length -
      indexesToRemove.size <
    2
  ) {
    window.alert(
      "Não é possível retirar estes tubos.\n\n" +
      "O caminho ficaria com menos de dois tubos. " +
      "Se já não precisares do caminho, utiliza Apagar.",
    );

    return;
  }

  const removedNodes =
    route.path.filter(
      (_node, nodeIndex) =>
        indexesToRemove.has(
          nodeIndex,
        ),
    );

  const removedNodeKeys =
    new Set(
      removedNodes.map(
        (node) =>
          nodeKey(node),
      ),
    );

  const removesDirectionNode =
    (
      route.directionStarts ?? []
    ).some(
      (node) =>
        removedNodeKeys.has(
          nodeKey(node),
        ),
    ) ||
    (
      route.directionEnds ?? []
    ).some(
      (node) =>
        removedNodeKeys.has(
          nodeKey(node),
        ),
    );

  const confirmationMessage =
    "Queres retirar " +
    indexesToRemove.size +
    ' tubo(s) do caminho "' +
    route.name +
    '"?\n\n' +
    "Os tubos não serão apagados do IFC. " +
    "Deixarão apenas de pertencer a este caminho." +
    (
      removesDirectionNode
        ? "\n\nAtenção: a seleção inclui tubos usados como início ou fim. " +
          "O sentido definido neste caminho será removido."
        : ""
    );

  const shouldRemove =
    window.confirm(
      confirmationMessage,
    );

  if (!shouldRemove) {
    flowMessage.value =
      "Remoção dos tubos cancelada.";

    return;
  }

  route.path =
    route.path.filter(
      (_node, nodeIndex) =>
        !indexesToRemove.has(
          nodeIndex,
        ),
    );

  if (removesDirectionNode) {
    delete route.directionStarts;
    delete route.directionEnds;
  }

  for (
    const [
      valveKey,
      linkedPipes,
    ] of valveControlledPipeLinks
  ) {
    const remainingLinks =
      linkedPipes.filter(
        (linkedPipe) =>
          !(
            linkedPipe.routeId ===
              route.id &&
            removedNodeKeys.has(
              nodeKey(linkedPipe),
            )
          ),
      );

    if (remainingLinks.length) {
      valveControlledPipeLinks.set(
        valveKey,
        remainingLinks,
      );
    } else {
      valveControlledPipeLinks.delete(
        valveKey,
      );

      valveBlockedPipeLinks.delete(
        valveKey,
      );
    }
  }

  for (
    const [
      valveKey,
      linkedPipes,
    ] of valveBlockedPipeLinks
  ) {
    const remainingBlockedLinks =
      linkedPipes.filter(
        (linkedPipe) =>
          !(
            linkedPipe.routeId ===
              route.id &&
            removedNodeKeys.has(
              nodeKey(linkedPipe),
            )
          ),
      );

    if (remainingBlockedLinks.length) {
      valveBlockedPipeLinks.set(
        valveKey,
        remainingBlockedLinks,
      );
    } else {
      valveBlockedPipeLinks.delete(
        valveKey,
      );
    }
  }

  for (const removedNode of removedNodes) {
    const adaptedRemovedNode =
      getAdaptedSavedRouteNodes({
        ...route,
        path: [removedNode],
      })[0];

    if (!adaptedRemovedNode) {
      continue;
    }

    const isUsedByAnotherRoute =
      savedRoutes.some(
        (otherRoute) =>
          otherRoute.id !== route.id &&
          routeContainsAdaptedNode(
            otherRoute,
            adaptedRemovedNode,
          ),
      );

    if (isUsedByAnotherRoute) {
      continue;
    }

    for (
      const circuitKey of
        getAllKnownCircuitKeys()
    ) {
      getAssignmentSet(
        circuitKey,
        adaptedRemovedNode.modelId,
      ).delete(
        adaptedRemovedNode.localId,
      );
    }

    hiddenFlowArrowElements
      .get(
        adaptedRemovedNode.modelId,
      )
      ?.delete(
        adaptedRemovedNode.localId,
      );

    reversedPipeDirections
      .get(
        adaptedRemovedNode.modelId,
      )
      ?.delete(
        adaptedRemovedNode.localId,
      );

    syncedPipeDirections
      .get(
        adaptedRemovedNode.modelId,
      )
      ?.delete(
        adaptedRemovedNode.localId,
      );
  }

  saveRoutesToStorage();
  saveValvePipeLinksToStorage();
  saveHiddenFlowArrowsToStorage();
  saveReversedDirectionsToStorage();
  saveSyncedPipeDirectionsToStorage();

  updateBlockedCount();
  updateManualStats();

  flowConnections.splice(0);

  await applyAllSavedRoutes();

  isFlowing.value = false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value = true;

  await finishEditedRouteChange();

flowMessage.value =
    indexesToRemove.size +
    ' tubo(s) retirado(s) do caminho "' +
    route.name +
    '".';
}

async function deleteSavedRoute(
  routeId: string,
  skipConfirmation = false,
  skipFinalRebuild = false,
) {
  const index = savedRoutes.findIndex(
    (route) =>
      route.id === routeId,
  );

  if (index === -1) {
    return;
  }

  const route = savedRoutes[index];

  if (route.locked) {
    flowMessage.value =
      'O caminho "' +
      route.name +
      '" está protegido. Desprotege primeiro para apagar.';

    return;
  }

let deleteConfirmationMessage =
  'Queres mesmo apagar o caminho "' +
  route.name +
  '"?\n\n' +
  "Esta ação é irreversível.";

if (route.mergeBackup) {
  deleteConfirmationMessage =
    'O caminho "' +
    route.name +
    '" foi criado através de uma união.\n\n' +
    "Se o apagares, o histórico necessário para " +
    "recuperar os caminhos anteriores também será eliminado.\n\n" +
    'Para os recuperar, cancela e usa "Desfazer união".\n\n' +
    "Queres mesmo apagar este caminho?";
}

if (!skipConfirmation) {
  const shouldDeleteRoute =
    window.confirm(
      deleteConfirmationMessage,
    );

  if (!shouldDeleteRoute) {
    flowMessage.value =
      'A eliminação do caminho "' +
      route.name +
      '" foi cancelada.';

    return;
  }
}


  if (
    highlightedSavedRouteId.value ===
    routeId
  ) {
    await modelHighlighter?.clear(
      "saved-route-highlight",
    );

    highlightedSavedRouteId.value =
      null;
  }

  savedRoutes.splice(
    index,
    1,
  );

  selectedRouteIdsForSimulation.delete(
    routeId,
  );

  selectedRouteIdsForGrouping.delete(
    routeId,
  );

  const exclusiveIdsByModel =
    new Map<string, number[]>();

  for (const node of route.path) {
    const isUsedByAnotherRoute =
      savedRoutes.some(
        (remainingRoute) =>
          remainingRoute.path.some(
            (remainingNode) =>
              isSameNode(
                remainingNode,
                node,
              ),
          ),
      );

    if (isUsedByAnotherRoute) {
      continue;
    }

    for (
      const circuit of
        getAllKnownCircuitKeys()
    ) {
      getAssignmentSet(
        circuit,
        node.modelId,
      ).delete(
        node.localId,
      );
    }

    reversedPipeDirections
      .get(node.modelId)
      ?.delete(node.localId);

    if (
      reversedPipeDirections
        .get(node.modelId)
        ?.size === 0
    ) {
      reversedPipeDirections.delete(
        node.modelId,
      );
    }

    syncedPipeDirections
      .get(node.modelId)
      ?.delete(node.localId);

    if (
      syncedPipeDirections
        .get(node.modelId)
        ?.size === 0
    ) {
      syncedPipeDirections.delete(
        node.modelId,
      );
    }

    const modelIds =
      exclusiveIdsByModel.get(
        node.modelId,
      ) ?? [];

    modelIds.push(
      node.localId,
    );

    exclusiveIdsByModel.set(
      node.modelId,
      modelIds,
    );
  }

  for (
    const [modelId, localIds] of
      exclusiveIdsByModel
  ) {
    const model =
      loadedModels.get(modelId);

    if (
      model &&
      localIds.length > 0
    ) {
      await model.resetHighlight(
        localIds,
      );
    }
  }

  flowConnections.splice(0);

  saveRoutesToStorage();
  saveReversedDirectionsToStorage();
  saveSyncedPipeDirectionsToStorage();

 updateManualStats();

if (!skipFinalRebuild) {
  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer(
      false,
    );
  } else {
    clearFlowVisuals();

    await fragmentManager.core.update(
      true,
    );
  }
}

  flowMessage.value =
    'O caminho "' +
    route.name +
    '" foi apagado. Os tubos pertencentes a outros caminhos foram mantidos.';
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
  isFlowing.value = false;

  isCentralSimulationRunning.value = false;

  isManualFlowAnimationRunning.value = false;

  isFlowManuallyPaused.value = true;

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer();

    isFlowing.value = false;
    return;
  }

  await fragmentManager.core.update(true);
}

function stopFlowForSavedRouteHighlight() {
  isFlowing.value = false;

  isCentralSimulationRunning.value = false;

  isManualFlowAnimationRunning.value = false;

  isFlowManuallyPaused.value = true;

  clearFlowVisuals(false);
}

async function clearFlowVisualsForRouteHighlight() {
  isApplyingSavedRouteHighlight.value =
    true;

  try {
    stopFlowForSavedRouteHighlight();

    selectedItems.clear();

    selectedCount.value =
      0;

    selectedIfcDetailsText.value =
      "";

    isIfcDetailsPanelOpen.value =
      false;

    selectedMepElementInfo.value =
      "Nenhum elemento classificado selecionado.";

    selectedTubeRouteInfo.value =
      "";

    if (modelHighlighter) {
      await modelHighlighter.clear(
        "select",
      );

      await modelHighlighter.clear(
        "saved-route-highlight",
      );
    }

    await clearPersistentCircuitHighlights();

    for (
      const model of
        loadedModels.values()
    ) {
      await model.resetHighlight();
    }

    await fragmentManager.core.update(
      true,
    );
  } finally {
    isApplyingSavedRouteHighlight.value =
      false;
  }
}

function getSavedRouteHighlightModelIdMap(
  route: SavedRoute,
) {
  const modelIdMap:
    Record<string, Set<number>> = {};

  const loadedModelIds = [
    ...loadedModels.keys(),
  ];

  const fallbackModelId =
    loadedModelIds[0];

  for (const node of route.path) {
    const modelId =
      loadedModels.has(node.modelId)
        ? node.modelId
        : fallbackModelId;

    if (!modelIdMap[modelId]) {
      modelIdMap[modelId] =
        new Set<number>();
    }

    modelIdMap[modelId].add(
      node.localId,
    );
  }

  return modelIdMap;
}

async function enforceActiveSavedRouteHighlight() {
  if (
    !highlightedSavedRouteId.value ||
    !modelHighlighter
  ) {
    return;
  }

  const route = savedRoutes.find(
    (savedRoute) =>
      savedRoute.id ===
      highlightedSavedRouteId.value,
  );

  if (!route) {
    return;
  }

  await clearFlowVisualsForRouteHighlight();

  const modelIdMap =
    getSavedRouteHighlightModelIdMap(
      route,
    );

  await modelHighlighter.clear(
    "saved-route-highlight",
  );

  await modelHighlighter.highlightByID(
    "saved-route-highlight",
    modelIdMap,
    true,
  );

  await fragmentManager.core.update(true);
}

async function highlightSavedRouteForEditing() {
  const selectedRoute =
    savedRoutes.find(
      (route) =>
        route.id ===
        selectedSavedRouteIdForEditing.value,
    );

  if (!selectedRoute) {
    if (
      highlightedSavedRouteId.value
    ) {
      const highlightedRoute =
        savedRoutes.find(
          (route) =>
            route.id ===
            highlightedSavedRouteId.value,
        );

      if (highlightedRoute) {
        await toggleSavedRouteHighlight(
          highlightedRoute,
        );
      }
    }

    return;
  }

  if (
    highlightedSavedRouteId.value ===
    selectedRoute.id
  ) {
    return;
  }

  await toggleSavedRouteHighlight(
    selectedRoute,
  );

  await rebuildManualFlowLayer();
}

async function toggleSavedRouteHighlight(
  route: SavedRoute,
) {
  if (
    !loadedModels.size ||
    !modelHighlighter
  ) {
    flowMessage.value =
      "Carrega primeiro o IFC antes de realçar um caminho.";
    return;
  }

  if (
    highlightedSavedRouteId.value ===
    route.id
  ) {
    isApplyingSavedRouteHighlight.value =
      true;

    try {
      await modelHighlighter.clear(
        "saved-route-highlight",
      );

      highlightedSavedRouteId.value =
        null;

      await restoreFlowVisualsAfterRouteHighlight();

      flowMessage.value =
        "Realce do caminho \"" +
        route.name +
        "\" removido.";
    } finally {
      isApplyingSavedRouteHighlight.value =
        false;
    }

    return;
  }

  isApplyingSavedRouteHighlight.value =
    true;

  try {
    await modelHighlighter.clear(
      "saved-route-highlight",
    );

    highlightedSavedRouteId.value =
  route.id;

stopFlowForSavedRouteHighlight();

await clearFlowVisualsForRouteHighlight();

    const modelIdMap =
      getSavedRouteHighlightModelIdMap(
        route,
      );

    await modelHighlighter.highlightByID(
      "saved-route-highlight",
      modelIdMap,
      true,
    );

    await fragmentManager.core.update(true);

    flowMessage.value =
      "Caminho \"" +
      route.name +
      "\" realçado.";
  } finally {
    isApplyingSavedRouteHighlight.value =
      false;
  }
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

async function createProtectedRouteDirections(
  route: SavedRoute,
) {
  const protectedDirections:
    ProtectedRouteDirection[] = [];

  for (const node of route.path) {
    const directionKey =
      automaticDirectionKey(
        route.temperature,
        node,
      );

    const automaticNeighbors =
      automaticDirectionNeighbors.get(
        directionKey,
      );

    let previous:
      FlowNode | null =
        automaticNeighbors?.previous
          ? {
              modelId:
                automaticNeighbors
                  .previous.modelId,

              localId:
                automaticNeighbors
                  .previous.localId,
            }
          : null;

    let next:
      FlowNode | null =
        automaticNeighbors?.next
          ? {
              modelId:
                automaticNeighbors
                  .next.modelId,

              localId:
                automaticNeighbors
                  .next.localId,
            }
          : null;

    if (
      !previous ||
      !next
    ) {
      for (
        const connection of
          flowConnections
      ) {
        if (
          connection.temperature !==
          route.temperature
        ) {
          continue;
        }

        if (
          !previous &&
          isSameNode(
            connection.to,
            node,
          )
        ) {
          previous = {
            modelId:
              connection.from.modelId,

            localId:
              connection.from.localId,
          };
        }

        if (
          !next &&
          isSameNode(
            connection.from,
            node,
          )
        ) {
          next = {
            modelId:
              connection.to.modelId,

            localId:
              connection.to.localId,
          };
        }
      }
    }

    protectedDirections.push({
      node: {
        modelId:
          node.modelId,

        localId:
          node.localId,
      },

      previous,

      next,

      reversed:
        isPipeDirectionReversed(
          node.modelId,
          node.localId,
        ),
    });
  }

  return protectedDirections;
}

function getProtectedDirectionConflict(
  route: SavedRoute,
  directions:
    ProtectedRouteDirection[],
) {
  const newDirectionsByNode =
    new Map(
      directions.map(
        (direction) => [
          nodeKey(
            direction.node,
          ),
          direction,
        ],
      ),
    );

  for (
    const otherRoute of
      savedRoutes
  ) {
    if (
      otherRoute.id === route.id ||
      !otherRoute.locked ||
      !otherRoute
        .protectedDirections
        ?.length
    ) {
      continue;
    }

    for (
      const otherDirection of
        otherRoute
          .protectedDirections
    ) {
      const newDirection =
        newDirectionsByNode.get(
          nodeKey(
            otherDirection.node,
          ),
        );

      if (!newDirection) {
        continue;
      }

      const oppositeNext =
        newDirection.next &&
        otherDirection.previous &&
        isSameNode(
          newDirection.next,
          otherDirection.previous,
        );

      const oppositePrevious =
        newDirection.previous &&
        otherDirection.next &&
        isSameNode(
          newDirection.previous,
          otherDirection.next,
        );

      if (
        oppositeNext ||
        oppositePrevious
      ) {
        return {
          otherRoute,
          node:
            newDirection.node,
        };
      }
    }
  }

  return null;
}

async function toggleSavedRouteProtection(
  routeId: string,
) {
  const route =
    savedRoutes.find(
      (savedRoute) =>
        savedRoute.id ===
        routeId,
    );

  if (!route) {
    return;
  }

  if (route.locked) {
    route.locked =
      false;

    delete route
      .protectedSnapshot;

    delete route
      .protectedDirections;

    saveRoutesToStorage();

    flowMessage.value =
      'Caminho "' +
      route.name +
      '" desprotegido. Já pode ser alterado.';

    return;
  }

  if (
    route
      .needsDirectionRedefinition
  ) {
    window.alert(
      "Não é possível proteger este caminho.\n\n" +
        "O sentido tem de ser definido novamente antes da proteção.",
    );

    return;
  }

  const protectedDirections =
    await createProtectedRouteDirections(
      route,
    );

  if (
    !protectedDirections.length
  ) {
    window.alert(
      "Não foi possível proteger este caminho.\n\n" +
        "Não existem sentidos válidos para congelar.",
    );

    return;
  }

  const conflict =
    getProtectedDirectionConflict(
      route,
      protectedDirections,
    );

  if (conflict) {
    window.alert(
      "Não foi possível proteger este caminho.\n\n" +
        "O tubo #" +
        conflict.node.localId +
        " tem um sentido oposto no caminho protegido \"" +
        conflict.otherRoute.name +
        "\".",
    );

    return;
  }

  route.protectedDirections =
    protectedDirections.map(
      cloneProtectedRouteDirection,
    );

  route.protectedSnapshot =
    createProtectedRouteSnapshot(
      route,
    );

  route.locked =
    true;

  selectedRouteIdsForGrouping.delete(
    route.id,
  );

  selectedSavedRouteIdForEditing.value =
    selectedSavedRouteIdForEditing.value ===
    route.id
      ? ""
      : selectedSavedRouteIdForEditing.value;

  if (
    selectedSavedRouteDirectionId.value ===
    route.id
  ) {
    await closeSavedRouteDirectionPanel(
      false,
    );
  }

  saveRoutesToStorage();

  flowMessage.value =
    'Caminho "' +
    route.name +
    '" completamente protegido. O percurso ficará congelado até ser desprotegido.';
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

function compareSavedRoutePaths(
  firstRoute: SavedRoute,
  secondRoute: SavedRoute,
): RouteComparisonResult {
  const firstNodeKeys =
    new Set(
      firstRoute.path.map(
        (node) => nodeKey(node),
      ),
    );

  const secondNodeKeys =
    new Set(
      secondRoute.path.map(
        (node) => nodeKey(node),
      ),
    );

  const firstOnlyNodes = [
    ...firstNodeKeys,
  ].filter(
    (nodeKeyValue) =>
      !secondNodeKeys.has(
        nodeKeyValue,
      ),
  );

  const secondOnlyNodes = [
    ...secondNodeKeys,
  ].filter(
    (nodeKeyValue) =>
      !firstNodeKeys.has(
        nodeKeyValue,
      ),
  );

  const commonNodeCount = [
    ...firstNodeKeys,
  ].filter(
    (nodeKeyValue) =>
      secondNodeKeys.has(
        nodeKeyValue,
      ),
  ).length;

  const totalUniqueNodeCount =
    new Set([
      ...firstNodeKeys,
      ...secondNodeKeys,
    ]).size;

  const overlapPercentage =
    totalUniqueNodeCount > 0
      ? Math.round(
          (
            commonNodeCount /
            totalUniqueNodeCount
          ) * 100,
        )
      : 0;

  const exactDuplicate =
    firstNodeKeys.size ===
      secondNodeKeys.size &&
    firstOnlyNodes.length === 0 &&
    secondOnlyNodes.length === 0;

  return {
    firstRoute,
    secondRoute,
    firstOnlyNodes,
    secondOnlyNodes,
    commonNodeCount,
    overlapPercentage,
    exactDuplicate,
  };
}

function compareSelectedSavedRoutes() {
  const selectedRoutes =
    savedRoutes.filter(
      (route) =>
        selectedRouteIdsForGrouping.has(
          route.id,
        ),
    );

  if (selectedRoutes.length !== 2) {
    flowMessage.value =
      "Seleciona exatamente dois percursos para comparar.";

    return;
  }

  const comparison =
    compareSavedRoutePaths(
      selectedRoutes[0],
      selectedRoutes[1],
    );

const firstRouteHasValveLinks =
  [
    ...valveControlledPipeLinks.values(),
  ].some(
    (linkedPipes) =>
      linkedPipes.some(
        (linkedPipe) =>
          linkedPipe.routeId ===
          comparison.firstRoute.id,
      ),
  );

const secondRouteHasValveLinks =
  [
    ...valveControlledPipeLinks.values(),
  ].some(
    (linkedPipes) =>
      linkedPipes.some(
        (linkedPipe) =>
          linkedPipe.routeId ===
          comparison.secondRoute.id,
      ),
  );

const firstRouteHasDirection =
  (
    comparison.firstRoute
      .directionStarts?.length ?? 0
  ) > 0 &&
  (
    comparison.firstRoute
      .directionEnds?.length ?? 0
  ) > 0;

const secondRouteHasDirection =
  (
    comparison.secondRoute
      .directionStarts?.length ?? 0
  ) > 0 &&
  (
    comparison.secondRoute
      .directionEnds?.length ?? 0
  ) > 0;

const comparisonMessage = [
  comparison.exactDuplicate
    ? "DUPLICADOS EXATOS"
    : "PERCURSOS DIFERENTES",

  "",

  comparison.firstRoute.name +
    ": " +
    comparison.firstRoute.path.length +
    " tubo(s)",

  comparison.secondRoute.name +
    ": " +
    comparison.secondRoute.path.length +
    " tubo(s)",

  "",

  "Tubos comuns: " +
    comparison.commonNodeCount,

  "Exclusivos do primeiro: " +
    comparison.firstOnlyNodes.length,

  "Exclusivos do segundo: " +
    comparison.secondOnlyNodes.length,

  "Sobreposição: " +
    comparison.overlapPercentage +
    "%",

  "",

  "CONFIGURAÇÃO DO PRIMEIRO",

  "Cor personalizada: " +
    (
      comparison.firstRoute.customColor ??
      "não"
    ),

  "Sentido definido: " +
    (
      firstRouteHasDirection
        ? "sim"
        : "não"
    ),

  "Protegido: " +
    (
      comparison.firstRoute.locked
        ? "sim"
        : "não"
    ),

  "Válvula associada: " +
    (
      firstRouteHasValveLinks
        ? "sim"
        : "não"
    ),

  "",

  "CONFIGURAÇÃO DO SEGUNDO",

  "Cor personalizada: " +
    (
      comparison.secondRoute.customColor ??
      "não"
    ),

  "Sentido definido: " +
    (
      secondRouteHasDirection
        ? "sim"
        : "não"
    ),

  "Protegido: " +
    (
      comparison.secondRoute.locked
        ? "sim"
        : "não"
    ),

  "Válvula associada: " +
    (
      secondRouteHasValveLinks
        ? "sim"
        : "não"
    ),
].join("\n");

  window.alert(
    comparisonMessage,
  );

  flowMessage.value =
    comparison.exactDuplicate
      ? "Os dois percursos selecionados são duplicados exatos."
      : "Os percursos foram comparados e têm diferenças.";
}

function nodeKey(node: FlowNode) {
  return `${node.modelId}:${node.localId}`;
}

function elementKey(modelId: string, localId: number) {
  return `${modelId}:${localId}`;
}

function saveMepElementsToStorage() {
  setActiveIfcStorageItem(
    MEP_ELEMENTS_STORAGE_KEY,
    JSON.stringify(mepElements),
  );
}

function saveRouteGroupsToStorage() {
  setActiveIfcStorageItem(
    ROUTE_GROUPS_STORAGE_KEY,
    JSON.stringify(
      savedRouteGroups,
    ),
  );
}

function loadRouteGroupsFromStorage() {
  const saved =
    getActiveIfcStorageItem(
      ROUTE_GROUPS_STORAGE_KEY,
    );

  if (!saved) {
    return;
  }

  try {
    const parsed =
      JSON.parse(
        saved,
      ) as SavedRouteGroup[];

    savedRouteGroups.splice(0);

    savedRouteGroups.push(
      ...parsed,
    );
  } catch (error) {
    console.error(
      "Erro ao carregar grupos de caminhos:",
      error,
    );
  }
}

function saveRoutesToStorage() {
  setActiveIfcStorageItem(
    ROUTES_STORAGE_KEY,
    JSON.stringify(savedRoutes),
  );
}

function loadRoutesFromStorage() {
  const saved =
    getActiveIfcStorageItem(
      ROUTES_STORAGE_KEY,
    );

  if (!saved) {
    return;
  }

  try {
    const parsed =
      JSON.parse(
        saved,
      ) as SavedRoute[];

    savedRoutes.splice(0);
    savedRoutes.push(...parsed);
  } catch (error) {
    console.error(
      "Erro ao carregar caminhos guardados:",
      error,
    );
  }
}

function loadMepElementsFromStorage() {
  const saved =
    getActiveIfcStorageItem(
      MEP_ELEMENTS_STORAGE_KEY,
    );

  if (!saved) {
    return;
  }

  try {
    const parsed =
      JSON.parse(saved) as Record<
        string,
        MepElement
      >;

    for (
      const [key, element] of
        Object.entries(parsed)
    ) {
      mepElements[key] = element;
    }
  } catch (error) {
    console.error(
      "Erro ao carregar elementos MEP guardados:",
      error,
    );
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

  setActiveIfcStorageItem(
  SYNCED_PIPE_DIRECTIONS_STORAGE_KEY,
  JSON.stringify(data),
);
}

function loadSyncedPipeDirectionsFromStorage() {
  const saved =
  getActiveIfcStorageItem(
    SYNCED_PIPE_DIRECTIONS_STORAGE_KEY,
  );

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

  setActiveIfcStorageItem(
    REVERSED_DIRECTIONS_STORAGE_KEY,
    JSON.stringify(data),
  );
}

function loadReversedDirectionsFromStorage() {
  const saved =
  getActiveIfcStorageItem(
    REVERSED_DIRECTIONS_STORAGE_KEY,
  );

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

function saveExcludedValvesToStorage() {
  setActiveIfcStorageItem(
    EXCLUDED_VALVES_STORAGE_KEY,
    JSON.stringify(
      [...excludedValveKeys],
    ),
  );
}

function loadExcludedValvesFromStorage() {
  const saved =
    getActiveIfcStorageItem(
      EXCLUDED_VALVES_STORAGE_KEY,
    );

  excludedValveKeys.clear();

  if (!saved) {
    return;
  }

  try {
    const parsed =
      JSON.parse(saved) as string[];

    for (const valveKey of parsed) {
      excludedValveKeys.add(
        valveKey,
      );
    }
  } catch (error) {
    console.error(
      "Erro ao carregar válvulas excluídas:",
      error,
    );
  }
}

function saveManualValveControlledPipesToStorage() {
  const serializedAssociations = [
    ...manualValveControlledPipes.entries(),
  ].map(
    (
      [
        valveKey,
        controlledPipes,
      ],
    ) => ({
      valveKey,

      controlledPipes:
        controlledPipes.map(
          (pipe) => ({
            modelId:
              pipe.modelId,

            localId:
              pipe.localId,
          }),
        ),
    }),
  );

  setActiveIfcStorageItem(
    MANUAL_VALVE_CONTROLLED_PIPES_STORAGE_KEY,
    JSON.stringify(
      serializedAssociations,
    ),
  );
}

function loadManualValveControlledPipesFromStorage() {
  manualValveControlledPipes.clear();

  const storedText =
    getActiveIfcStorageItem(
      MANUAL_VALVE_CONTROLLED_PIPES_STORAGE_KEY,
    );

  if (!storedText) {
    return;
  }

  try {
    const storedAssociations =
      JSON.parse(
        storedText,
      ) as Array<{
        valveKey: string;

        controlledPipes:
          ManualValveControlledPipe[];
      }>;

    for (
      const association of
        storedAssociations
    ) {
      if (
        !association.valveKey ||
        !Array.isArray(
          association.controlledPipes,
        )
      ) {
        continue;
      }

      manualValveControlledPipes.set(
        association.valveKey,
        association.controlledPipes.map(
          (pipe) => ({
            modelId:
              String(pipe.modelId),

            localId:
              Number(pipe.localId),
          }),
        ).filter(
          (pipe) =>
            pipe.modelId &&
            Number.isFinite(
              pipe.localId,
            ),
        ),
      );
    }
  } catch (error) {
    console.error(
      "Erro ao carregar os tubos controlados manualmente pelas válvulas:",
      error,
    );

    manualValveControlledPipes.clear();
  }
}

function saveValvePipeLinksToStorage() {
  const data = [...valveControlledPipeLinks.entries()].map(
    ([valveKey, linkedPipes]) => ({
      valveKey,
      linkedPipes,
    }),
  );

  setActiveIfcStorageItem(
  VALVE_PIPE_LINKS_STORAGE_KEY,
  JSON.stringify(data),
);
}

function loadValvePipeLinksFromStorage() {
  const saved =
  getActiveIfcStorageItem(
    VALVE_PIPE_LINKS_STORAGE_KEY,
  );

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

function savePipeTypeFlowNodesToStorage() {
  setActiveIfcStorageItem(
    PIPE_TYPE_FLOW_NODES_STORAGE_KEY,
    JSON.stringify(
      [...pipeTypeFlowNodes],
    ),
  );
}

function loadPipeTypeFlowNodesFromStorage() {
  const saved =
  getActiveIfcStorageItem(
    PIPE_TYPE_FLOW_NODES_STORAGE_KEY,
  );

  if (!saved) {
    return;
  }

  try {
    const parsed =
      JSON.parse(saved) as string[];

    pipeTypeFlowNodes.clear();

    for (const nodeKeyValue of parsed) {
      pipeTypeFlowNodes.add(nodeKeyValue);
    }
  } catch (error) {
    console.error(
      "Erro ao carregar elementos Pipe Types:",
      error,
    );
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

  setActiveIfcStorageItem(
  HIDDEN_FLOW_ARROWS_STORAGE_KEY,
  JSON.stringify(data),
);
}

function loadHiddenFlowArrowsFromStorage() {
  const saved =
  getActiveIfcStorageItem(
    HIDDEN_FLOW_ARROWS_STORAGE_KEY,
  );

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

const currentState =
  isValveElementType(elementType)
    ? (
        mepElements[key]?.state ===
          "closed"
          ? "closed"
          : "open"
      )
    : mepElements[key]?.state;

      const isManualValve =
        isValveElementType(
          elementType,
        );

      const existingElement =
        mepElements[key];

      mepElements[key] = {
        ...existingElement,
        modelId,
        localId,
        elementType,
        circuitType:
          existingElement?.circuitType ??
          "unknown",
        state: currentState,
        isShutoffValve:
          isManualValve
            ? true
            : existingElement
                ?.isShutoffValve,
        valveIdentificationText:
          isManualValve
            ? existingElement
                ?.valveIdentificationText ||
              "Definição manual"
            : existingElement
                ?.valveIdentificationText,
      };
    }
  }

  saveMepElementsToStorage();

if (
  elementType === "normallyClosedValve" ||
  elementType === "normallyOpenValve"
) {
  flowMessage.value =
    `${selectedCount.value} elemento(s) definidos como ${getElementTypeLabel(elementType)}. ` +
    "A indicação NA/NF é apenas informativa. " +
    "O estado atual foi mantido ou iniciado como aberto.";

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
removeActiveIfcStorageItem(
  VALVE_PIPE_LINKS_STORAGE_KEY,
);

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

function getAttributeValueText(
  value: any,
): string {
  if (
    value === null ||
    value === undefined
  ) {
    return "";
  }

  if (
    typeof value === "string" ||
    typeof value === "number"
  ) {
    return String(value);
  }

  if (
    typeof value === "object"
  ) {
    if ("value" in value) {
      return getAttributeValueText(
        value.value,
      );
    }

    if ("Value" in value) {
      return getAttributeValueText(
        value.Value,
      );
    }

    if ("name" in value) {
      return getAttributeValueText(
        value.name,
      );
    }

    if ("Name" in value) {
      return getAttributeValueText(
        value.Name,
      );
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

function getValveManagementOptions() {
  return Object.values(
    mepElements,
  )
    .filter((element) => {
      if (
        !isValveElementType(
          element.elementType,
        ) ||
        element.isShutoffValve !== true
      ) {
        return false;
      }

      const key = elementKey(
        element.modelId,
        element.localId,
      );

      return !excludedValveKeys.has(
        key,
      );
    })
    .map((element) => {
      const key = elementKey(
        element.modelId,
        element.localId,
      );

const originalName =
  valveOriginalDesignations[key] ||
  element.valveIdentificationText ||
  `Válvula #${element.localId}`;

const displayName =
  element.name?.trim() ||
  element.systemName?.trim() ||
  originalName;

const normalStateLabel =
  element.elementType ===
  "normallyClosedValve"
    ? "NF"
    : "NA";
return {
  key,

  modelId:
    element.modelId,

  localId:
    element.localId,

  label:
    displayName +
    " · #" +
    element.localId +
    " · " +
    normalStateLabel,
};
    })
    .sort(
      (
        firstOption,
        secondOption,
      ) =>
        firstOption.label.localeCompare(
          secondOption.label,
        ),
    );
}

function isValveSelectedForManagement(
  valveKey: string,
) {
  return (
    selectedValveKeysForManagement.has(
      valveKey,
    )
  );
}

function selectAllManagedValves() {
  selectedValveKeysForManagement.clear();

  for (
    const valveOption of
      getValveManagementOptions()
  ) {
    selectedValveKeysForManagement.add(
      valveOption.key,
    );
  }

  flowMessage.value =
    selectedValveKeysForManagement.size +
    " válvula(s) selecionada(s).";
}

function areAllManagedValvesSelected() {
  const valveOptions =
    getValveManagementOptions();

  return (
    valveOptions.length > 0 &&
    valveOptions.every(
      (valveOption) =>
        selectedValveKeysForManagement.has(
          valveOption.key,
        ),
    )
  );
}

function toggleValveManagementSelection(
  valveKey: string,
) {
  if (
    selectedValveKeysForManagement.has(
      valveKey,
    )
  ) {
    selectedValveKeysForManagement.delete(
      valveKey,
    );

    return;
  }

  selectedValveKeysForManagement.add(
    valveKey,
  );
}

async function clearManagedValveHighlight() {
  const idsByModel =
    new Map<string, number[]>();

  for (
    const valveKey of
      highlightedValveKeysForManagement
  ) {
    const valveNode =
      getValveNodeFromDesignationKey(
        valveKey,
      );

    if (!valveNode) {
      continue;
    }

    const modelIds =
      idsByModel.get(
        valveNode.modelId,
      ) ?? [];

    modelIds.push(
      valveNode.localId,
    );

    idsByModel.set(
      valveNode.modelId,
      modelIds,
    );
  }

  for (
    const [modelId, localIds] of
      idsByModel
  ) {
    const model =
      loadedModels.get(modelId);

    if (
      !model ||
      localIds.length === 0
    ) {
      continue;
    }

    await model.resetHighlight(
      localIds,
    );
  }

  highlightedValveKeysForManagement.clear();

  await fragmentManager.core.update(
    true,
  );
}

async function highlightSelectedManagedValves() {
  if (
    selectedValveKeysForManagement.size === 0
  ) {
    flowMessage.value =
      "Seleciona primeiro uma ou mais válvulas.";

    return;
  }

  await clearManagedValveHighlight();

  const idsByModel =
    new Map<string, number[]>();

  for (
    const valveKey of
      selectedValveKeysForManagement
  ) {
    const valveNode =
      getValveNodeFromDesignationKey(
        valveKey,
      );

    if (!valveNode) {
      continue;
    }

    const modelIds =
      idsByModel.get(
        valveNode.modelId,
      ) ?? [];

    modelIds.push(
      valveNode.localId,
    );

    idsByModel.set(
      valveNode.modelId,
      modelIds,
    );

    highlightedValveKeysForManagement.add(
      valveKey,
    );
  }

  let highlightedCount = 0;

  for (
    const [modelId, localIds] of
      idsByModel
  ) {
    const model =
      loadedModels.get(modelId);

    if (
      !model ||
      localIds.length === 0
    ) {
      continue;
    }

    await model.highlight(
      localIds,
      createHighlight(
        0xff00ff,
        "managed-valves-highlight",
      ),
    );

    highlightedCount +=
      localIds.length;
  }

  await fragmentManager.core.update(
    true,
  );

  flowMessage.value =
    highlightedCount +
    " válvula(s) realçada(s).";
}

async function setSelectedManagedValvesState(
  targetState: "open" | "closed",
) {
  if (
    selectedValveKeysForManagement.size === 0
  ) {
    flowMessage.value =
      "Seleciona primeiro uma ou mais válvulas.";

    return;
  }

  const previousDesignationKey =
    selectedValveDesignationKey.value;

  let changedCount = 0;

  for (
    const valveKey of
      selectedValveKeysForManagement
  ) {
    const valveNode =
      getValveNodeFromDesignationKey(
        valveKey,
      );

    if (!valveNode) {
      continue;
    }

    const element =
      mepElements[
        elementKey(
          valveNode.modelId,
          valveNode.localId,
        )
      ];

    if (
      !element ||
      !isConfirmedShutoffValve(
        element,
      )
    ) {
      continue;
    }

    if (
      element.state === targetState
    ) {
      continue;
    }

    selectedValveDesignationKey.value =
      valveKey;

    await setSelectedValvesState(
      targetState,
    );

    changedCount++;
  }

  selectedValveDesignationKey.value =
    previousDesignationKey;

  saveMepElementsToStorage();

  flowMessage.value =
    changedCount > 0
      ? changedCount +
        (
          targetState === "open"
            ? " válvula(s) aberta(s)."
            : " válvula(s) fechada(s)."
        )
      : (
          targetState === "open"
            ? "As válvulas selecionadas já estavam abertas."
            : "As válvulas selecionadas já estavam fechadas."
        );
}

async function excludeSelectedManagedValves() {
  if (
    selectedValveKeysForManagement.size === 0
  ) {
    flowMessage.value =
      "Seleciona primeiro uma ou mais válvulas.";

    return;
  }

  const selectedValveCount =
    selectedValveKeysForManagement.size;

  const shouldExclude =
    window.confirm(
      "Tens a certeza de que queres eliminar " +
        selectedValveCount +
        " válvula(s) da configuração?\n\n" +
        "As válvulas detetadas automaticamente voltarão " +
        "a aparecer quando repetires o scan do IFC.",
    );

  if (!shouldExclude) {
    flowMessage.value =
      "Eliminação das válvulas cancelada.";

    return;
  }

  await clearManagedValveHighlight();

  for (
    const valveKey of
      selectedValveKeysForManagement
  ) {
    const valveNode =
      getValveNodeFromDesignationKey(
        valveKey,
      );

    const valveElement =
      valveNode
        ? mepElements[
            elementKey(
              valveNode.modelId,
              valveNode.localId,
            )
          ]
        : undefined;

    const isManualValve =
      valveElement
        ?.valveIdentificationText ===
      "Definição manual";

    if (isManualValve) {
      delete mepElements[
        valveKey
      ];

      delete valveOriginalDesignations[
        valveKey
      ];
    } else {
      excludedValveKeys.add(
        valveKey,
      );
    }

    const linkedPipes =
      valveControlledPipeLinks.get(
        valveKey,
      ) ??
      valveBlockedPipeLinks.get(
        valveKey,
      ) ??
      [];

    restoreValveLinkedPipesToOriginalCircuit(
      linkedPipes,
    );

    for (const pipeNode of linkedPipes) {
      unblockPipeForRoute(
        pipeNode.routeId,
        pipeNode,
      );
    }

    valveControlledPipeLinks.delete(
      valveKey,
    );

    valveBlockedPipeLinks.delete(
      valveKey,
    );
  }

  saveExcludedValvesToStorage();
  saveValvePipeLinksToStorage();
  saveMepElementsToStorage();

  selectedValveKeysForManagement.clear();

  selectedValveDesignationKey.value =
    "";

  selectedValveDesignation.value =
    "nenhuma válvula selecionada";

  selectedValveOriginalDesignation.value =
    "";

  pendingValveDesignation.value =
    "";

  selectedValveForPipeLink.value =
    null;

  updateBlockedCount();

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value =
    selectedValveCount +
    " válvula(s) eliminada(s) da configuração.";
}

function clearValveManagementSelection() {
  selectedValveKeysForManagement.clear();
}

function toggleValveManagementPanel() {
  isValveManagementPanelOpen.value =
    !isValveManagementPanelOpen.value;
}

function getValveDesignationOptions() {
  return getValveManagementOptions().map(
    (valveOption) => ({
      key:
        valveOption.key,

      label:
        valveOption.label,
    }),
  );
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

  const suggestedRoute =
  getSingleAutomaticRouteForValve(
    element,
  );

if (suggestedRoute) {
  selectedValveAssociationRouteId.value =
    suggestedRoute.id;
} else {
  selectedValveAssociationRouteId.value =
    "";
}
}

async function clearSelectedValveFromDropdown(
  shouldRebuildFlow = true,
) {

  if (
    hasManualValvePipeDraftChanges.value
  ) {
    const shouldClear =
      window.confirm(
        "Existem alterações no rascunho da válvula que ainda não foram guardadas.\n\n" +
          "Queres sair e perder essas alterações?",
      );

    if (!shouldClear) {
      return false;
    }
  }

  isValveFocusModeActive.value =
  false;

  isManualValvePipeDefinitionMode.value =
    false;

  hasManualValvePipeDraftChanges.value =
    false;

  manualValvePipeDraft.value =
    [];

  await clearManualValvePipeDraftHighlight();

  if (
    highlightedValveFromDropdown.value
  ) {
    const previousModel =
      loadedModels.get(
        highlightedValveFromDropdown.value
          .modelId,
      );

    if (previousModel) {
      await previousModel.resetHighlight([
        highlightedValveFromDropdown.value
          .localId,
      ]);
    }
  }

  highlightedValveFromDropdown.value =
    null;

  selectedValveDesignationKey.value =
    "";

  selectedValveDesignation.value =
    "nenhuma válvula selecionada";

  selectedValveOriginalDesignation.value =
    "";

  pendingValveDesignation.value =
    "";

  selectedValveForPipeLink.value =
    null;

  selectedValveAssociationRouteId.value =
    "";

  selectedValveSwitchMode.value =
    "none";

  selectedItems.clear();

  selectedCount.value =
    0;

  isValveDesignationDropdownOpen.value =
    false;

  if (shouldRebuildFlow) {
  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer();
  } else {
    await fragmentManager.core.update(
      true,
    );
  }
}

  isFlowing.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isManualFlowAnimationRunning.value =
    false;

  isFlowManuallyPaused.value =
    true;

  flowMessage.value =
    "Nenhuma válvula selecionada. Clica em Atualizar para preparar a simulação.";

  return true;
}

async function selectValveDesignationOption(
  valveKey: string,
) {
  if (!valveKey) {
    await clearSelectedValveFromDropdown();

    return;
  }

  selectedValveDesignationKey.value =
    valveKey;

  isValveDesignationDropdownOpen.value =
    false;

  await selectValveDesignationFromDropdown();
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

  selectedValveAssociationRouteId.value =
  "";

selectedValveSwitchMode.value =
  "none";

  flowMessage.value =
    "Válvula selecionada no dropdown e destacada no modelo.";
}

function getValveNodeForDesignationEditing() {
  return getValveNodeForControl();
}

function normalizeIfcValue(value: any, depth = 0): any {
  if (depth > 8) {
    return "[limite de profundidade]";
  }

  if (value === null || value === undefined) {
    return value;
  }

  if (
    typeof value === "string" ||
    typeof value === "number" ||
    typeof value === "boolean"
  ) {
    return value;
  }

  if (Array.isArray(value)) {
    return value.map((item) => normalizeIfcValue(item, depth + 1));
  }

  if (typeof value === "object") {
    const result: Record<string, any> = {};

    for (const [key, nestedValue] of Object.entries(value)) {
      result[key] = normalizeIfcValue(nestedValue, depth + 1);
    }

    return result;
  }

  return String(value);
}

function collectRevitFamilyAndTypeValues(data: any) {
  const results = {
    family: "não encontrado",
    type: "não encontrado",
    candidatosEncontrados: [] as any[],
  };

  const seen = new Set<string>();

  function normalizeText(text: string) {
    return String(text)
      .toLowerCase()
      .normalize("NFD")
      .replace(/[\u0300-\u036f]/g, "")
      .replace(/\s+/g, "")
      .replace(/_/g, "")
      .replace(/-/g, "");
  }

  function getSimpleValue(value: any): any {
    if (value === null || value === undefined) {
      return null;
    }

    if (
      typeof value === "string" ||
      typeof value === "number" ||
      typeof value === "boolean"
    ) {
      return value;
    }

    if (typeof value === "object") {
      if ("value" in value) return getSimpleValue(value.value);
      if ("Value" in value) return getSimpleValue(value.Value);
      if ("wrappedValue" in value) return getSimpleValue(value.wrappedValue);
      if ("NominalValue" in value) return getSimpleValue(value.NominalValue);
      if ("nominalValue" in value) return getSimpleValue(value.nominalValue);
      if ("Name" in value) return getSimpleValue(value.Name);
      if ("name" in value) return getSimpleValue(value.name);
    }

    return null;
  }

  function addCandidate(
    category: "family" | "type",
    pathText: string,
    fieldName: string,
    value: any,
  ) {
    const simpleValue = getSimpleValue(value);

    if (
      simpleValue === null ||
      simpleValue === undefined ||
      String(simpleValue).trim() === ""
    ) {
      return;
    }

    const uniqueKey =
      category + "|" + pathText + "|" + fieldName + "|" + String(simpleValue);

    if (seen.has(uniqueKey)) {
      return;
    }

    seen.add(uniqueKey);

    results.candidatosEncontrados.push({
      categoria: category,
      caminho: pathText,
      campo: fieldName,
      valor: simpleValue,
    });

    if (results[category] === "não encontrado") {
      results[category] = String(simpleValue);
    }
  }

  function classifyField(text: string): "family" | "type" | null {
    const normalizedText = normalizeText(text);

    if (
      normalizedText.includes("family") ||
      normalizedText.includes("SystemFamily") ||
      normalizedText.includes("familyname") ||
      normalizedText.includes("revitfamily")
    ) {
      return "family";
    }

    if (
  normalizedText.includes("typename") ||
  normalizedText.includes("objecttype") ||
  normalizedText.includes("revittypename") ||
  normalizedText.includes("revittypemark") ||
  normalizedText.includes("familyandtype") ||
  normalizedText.includes("familytype") ||
  normalizedText.includes("typesymbol") ||
  normalizedText.includes("typecatalog")
) {
  return "type";
}

    return null;
  }

  function visit(value: any, path: string[] = []) {
    if (value === null || value === undefined) {
      return;
    }

    const pathText = path.join(".");
    const lastKey = path[path.length - 1] || "";

    const pathCategory = classifyField(pathText);
    const keyCategory = classifyField(lastKey);

    if (pathCategory) {
      addCandidate(pathCategory, pathText, lastKey, value);
    }

    if (keyCategory) {
      addCandidate(keyCategory, pathText, lastKey, value);
    }

    if (typeof value === "object" && !Array.isArray(value)) {
      const objectName = String(
        getSimpleValue(value.Name) ??
        getSimpleValue(value.name) ??
        "",
      );

      const objectCategory = classifyField(objectName);

      if (objectCategory) {
        addCandidate(objectCategory, pathText, objectName, value);
      }
    }

    if (Array.isArray(value)) {
      value.forEach((item, index) => {
        visit(item, path.concat(String(index)));
      });

      return;
    }

    if (typeof value === "object") {
      for (const [key, nestedValue] of Object.entries(value)) {
        visit(nestedValue, path.concat(key));
      }
    }
  }

  visit(data);

  return results;
}

function findIfcPropertyValue(
  data: any,
  propertyNames: string[],
) {
  const normalizedNames = propertyNames.map((name) =>
    name
      .toLowerCase()
      .normalize("NFD")
      .replace(/[\u0300-\u036f]/g, "")
      .replace(/[\s_-]/g, ""),
  );

  function normalizeName(value: any) {
    return getAttributeValueText(value)
      .toLowerCase()
      .normalize("NFD")
      .replace(/[\u0300-\u036f]/g, "")
      .replace(/[\s_-]/g, "");
  }

  function getPropertyValue(value: any): string {
    if (value === null || value === undefined) {
      return "";
    }

    if (
      typeof value === "string" ||
      typeof value === "number" ||
      typeof value === "boolean"
    ) {
      return String(value).trim();
    }

    if (typeof value === "object") {
      const possibleValues = [
        value.NominalValue,
        value.nominalValue,
        value.Value,
        value.value,
        value.wrappedValue,
      ];

      for (const possibleValue of possibleValues) {
        const extractedValue =
          getAttributeValueText(possibleValue).trim();

        if (extractedValue) {
          return extractedValue;
        }
      }
    }

    return "";
  }

  function visit(value: any): string | null {
    if (value === null || value === undefined) {
      return null;
    }

    if (Array.isArray(value)) {
      for (const item of value) {
        const result = visit(item);

        if (result) {
          return result;
        }
      }

      return null;
    }

    if (typeof value !== "object") {
      return null;
    }

    const propertyName = normalizeName(
      value.Name ?? value.name,
    );

    if (
      propertyName &&
      normalizedNames.includes(propertyName)
    ) {
      const propertyValue = getPropertyValue(value);

      if (propertyValue) {
        return propertyValue;
      }
    }

    for (const [key, nestedValue] of Object.entries(value)) {
      const normalizedKey = normalizeName(key);

      if (normalizedNames.includes(normalizedKey)) {
        const directValue = getPropertyValue(nestedValue);

        if (directValue) {
          return directValue;
        }
      }

      const nestedResult = visit(nestedValue);

      if (nestedResult) {
        return nestedResult;
      }
    }

    return null;
  }

  return visit(data) ?? "não encontrado";
}

function getSingleDiameterValue(value: string) {
  if (!value || value === "não encontrado") {
    return "não encontrado";
  }

  const numericMatch = value.match(/\d+(?:[.,]\d+)?/);

  if (!numericMatch) {
    return value;
  }

  const numericDiameter = Number(
    numericMatch[0].replace(",", "."),
  );

  const diameterInMillimeters =
    numericDiameter <= 1
      ? numericDiameter * 1000
      : numericDiameter;

  return (
    "DN " +
    Number(diameterInMillimeters.toFixed(2))
  );
}

function findMechanicalSystemName(data: any) {
  function normalizeText(value: any) {
    return getAttributeValueText(value)
      .trim()
      .toLowerCase()
      .replace(/\s+/g, "")
      .replace(/_/g, "");
  }

  function extractValue(value: any) {
    if (value === null || value === undefined) {
      return "";
    }

    if (
      typeof value === "string" ||
      typeof value === "number"
    ) {
      return String(value).trim();
    }

    const possibleValues = [
      value.NominalValue,
      value.nominalValue,
      value.Value,
      value.value,
      value.wrappedValue,
    ];

    for (const possibleValue of possibleValues) {
      const text =
        getAttributeValueText(possibleValue).trim();

      if (text) {
        return text;
      }
    }

    return "";
  }

  function visit(
    value: any,
    insideMechanical = false,
  ): string {
    if (
      value === null ||
      value === undefined ||
      typeof value !== "object"
    ) {
      return "";
    }

    const objectName = normalizeText(
      value.Name ?? value.name,
    );

    const isMechanical =
      insideMechanical ||
      objectName === "mechanical";

    if (
      isMechanical &&
      objectName === "systemname"
    ) {
      return extractValue(value);
    }

    if (Array.isArray(value)) {
      for (const item of value) {
        const result = visit(item, isMechanical);

        if (result) {
          return result;
        }
      }

      return "";
    }

    for (const [key, nestedValue] of Object.entries(value)) {
      const normalizedKey = normalizeText(key);

      const nextInsideMechanical =
        isMechanical ||
        normalizedKey === "mechanical";

      if (
        nextInsideMechanical &&
        normalizedKey === "systemname"
      ) {
        const directValue = extractValue(nestedValue);

        if (directValue) {
          return directValue;
        }
      }

      const result = visit(
        nestedValue,
        nextInsideMechanical,
      );

      if (result) {
        return result;
      }
    }

    return "";
  }

  return visit(data);
}

function findRevitTypeValue(data: any) {
  function normalizeText(value: any) {
    return getAttributeValueText(value)
      .trim()
      .toLowerCase()
      .replace(/\s+/g, "")
      .replace(/_/g, "");
  }

  function extractPropertyValue(value: any) {
    if (!value || typeof value !== "object") {
      return "";
    }

    const possibleValues = [
      value.NominalValue,
      value.nominalValue,
      value.Value,
      value.value,
      value.wrappedValue,
    ];

    for (const possibleValue of possibleValues) {
      const text =
        getAttributeValueText(possibleValue).trim();

      if (text) {
        return text;
      }
    }

    return "";
  }

  function visit(value: any): string {
    if (
      value === null ||
      value === undefined ||
      typeof value !== "object"
    ) {
      return "";
    }

    if (Array.isArray(value)) {
      for (const item of value) {
        const result = visit(item);

        if (result) {
          return result;
        }
      }

      return "";
    }

    const propertyName = normalizeText(
      value.Name ?? value.name,
    );

    if (propertyName === "type") {
      const propertyValue =
        extractPropertyValue(value);

      if (propertyValue) {
        return propertyValue;
      }
    }

    for (const [key, nestedValue] of Object.entries(value)) {
      if (normalizeText(key) === "type") {
        const directValue =
          getAttributeValueText(nestedValue).trim();

        if (
          directValue &&
          !directValue.toLowerCase().startsWith("ifc")
        ) {
          return directValue;
        }
      }

      const result = visit(nestedValue);

      if (result) {
        return result;
      }
    }

    return "";
  }

  return visit(data);
}

async function extractSelectedIfcInformation() {
  const selectedNode = getFirstSelectedNode();

  if (!selectedNode) {
    flowMessage.value =
      "Seleciona primeiro um tubo ou elemento no modelo.";
    return;
  }

  const model = loadedModels.get(selectedNode.modelId);

  if (!model) {
    flowMessage.value =
      "Modelo do elemento selecionado não encontrado.";
    return;
  }

  const itemData = await model.getItemsData(
    [selectedNode.localId],
    {
      attributesDefault: true,
      relations: {
  IsTypedBy: {
    attributes: true,
    relations: true,
  },
  DefinesOccurrence: {
    attributes: true,
    relations: true,
  },
  IsDefinedBy: {
    attributes: true,
    relations: true,
  },
  HasAssignments: {
  attributes: true,
  relations: false,
},
},
    },
  );

  const itemEntity: any = Array.isArray(itemData)
    ? itemData[0]
    : undefined;

  const rawIfcData = normalizeIfcValue(itemEntity);

  const revitElementId =
  getAttributeValueText(
    itemEntity?.Tag ??
    itemEntity?.tag,
  ).trim() ||
  findIfcPropertyValue(
    rawIfcData,
    [
      "Tag",
      "BATID",
      "Element ID",
      "ElementId",
    ],
  );

  const revitValues =
    collectRevitFamilyAndTypeValues(rawIfcData);

  const exactRevitType =
  findRevitTypeValue(rawIfcData);

  const systemType = findIfcPropertyValue(
  rawIfcData,
  [
    "System Type",
    "SystemType",
    "System Classification",
    "SystemClassification",
  ],
);

  const mechanicalSystemName =
  findMechanicalSystemName(rawIfcData);

const fallbackSystemName =
  findIfcPropertyValue(
    rawIfcData,
    [
      "System Name",
      "SystemName",
      "System Abbreviation",
      "SystemAbbreviation",
    ],
  );

const systemName =
  mechanicalSystemName ||
  fallbackSystemName;

const diameterValue = findIfcPropertyValue(
  rawIfcData,
  [
    "Diameter",
    "Nominal Diameter",
    "NominalDiameter",
    "Outer Diameter",
    "OuterDiameter",
    "Inner Diameter",
    "InnerDiameter",
    "Size",
  ],
);

  const stateValue = findIfcPropertyValue(
  rawIfcData,
  [
    "State",
    "Estado",
    "Valve State",
    "ValveState",
  ],
);

  const typeRelation =
    itemEntity?.IsTypedBy?.[0] ??
    itemEntity?.isTypedBy?.[0] ??
    null;

  let revitFamily = revitValues.family;

  if (typeRelation) {
    const typeLocalId = Number(
      typeRelation._localId?.value ??
      typeRelation.localId?.value ??
      typeRelation._localId ??
      typeRelation.localId,
    );

    if (Number.isFinite(typeLocalId)) {
      const typeItemData = await model.getItemsData(
  [typeLocalId],
  {
    attributesDefault: false,
    attributes: [
      "Name",
      "Description",
      "ElementType",
      "ApplicableOccurrence",
      "Tag",
      "PredefinedType",
      "ObjectType",
    ],
    relationsDefault: {
      attributes: false,
      relations: false,
    },
  },
);

      const typeEntity: any = Array.isArray(typeItemData)
        ? typeItemData[0]
        : undefined;

      console.log(
        "ENTIDADE IFC DE TIPO COMPLETA:",
        typeEntity,
      );

      const possibleFamilyValues = [
        typeEntity?.FamilyName,
        typeEntity?.familyName,
        typeEntity?.Family,
        typeEntity?.family,
        typeEntity?.ElementType,
        typeEntity?.elementType,
        typeEntity?.Name,
        typeEntity?.name,
      ];

      for (
        const possibleFamilyValue of possibleFamilyValues
      ) {
        const familyText =
          getAttributeValueText(
            possibleFamilyValue,
          ).trim();

        if (familyText) {
          revitFamily = familyText;
          break;
        }
      }
    }
  }

  const extractedData = {
  elementoSelecionado: {
    modelId: selectedNode.modelId,
    localId: selectedNode.localId,
  },
  "Revit Element ID": revitElementId,
  Family: revitFamily,
  Type:
  exactRevitType ||
  revitValues.type,
  "System Type": systemType,
  "System Name": systemName,
    Diameter:
    getSingleDiameterValue(
      diameterValue,
    ),

  State: stateValue,

};

  selectedIfcDetailsText.value =
    JSON.stringify(extractedData, null, 2);

  isIfcDetailsPanelOpen.value = true;

  console.log(
    "Family e Type extraídos:",
    extractedData,
  );

  flowMessage.value =
    "Family e Type extraídos para o elemento #" +
    selectedNode.localId +
    ".";
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

function countMepElementsByType(
  elementType: MepElementType,
) {
  return Object.values(
    mepElements,
  ).filter((element) => {
    if (
      elementType ===
        "normallyOpenValve" ||
      elementType ===
        "normallyClosedValve"
    ) {
      return (
        element.elementType ===
          elementType &&
        isConfirmedShutoffValve(
          element,
        )
      );
    }

    return (
      element.elementType ===
      elementType
    );
  }).length;
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
  return Object.values(
    mepElements,
  ).filter(
    (element) =>
      isConfirmedShutoffValve(
        element,
      ),
  ).length;
}

function countReservoirElements() {
  return (
    countMepElementsByType("reservoirWithResistance") +
    countMepElementsByType("reservoirWithoutResistance")
  );
}

function getMepElementIdsByType(
  elementType: MepElementType,
) {
  const idsByModel =
    new Map<string, number[]>();

  const mustBeConfirmedShutoffValve =
    elementType ===
      "normallyOpenValve" ||
    elementType ===
      "normallyClosedValve" ||
    elementType ===
      "isolationValve";

  for (
    const element of
      Object.values(mepElements)
  ) {
    if (
      element.elementType !==
      elementType
    ) {
      continue;
    }

    if (
      mustBeConfirmedShutoffValve &&
      !isConfirmedShutoffValve(
        element,
      )
    ) {
      continue;
    }

    if (
      !idsByModel.has(
        element.modelId,
      )
    ) {
      idsByModel.set(
        element.modelId,
        [],
      );
    }

    idsByModel
      .get(element.modelId)
      ?.push(element.localId);
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

async function clearValveTypeHighlight() {
  for (
    const model of
      loadedModels.values()
  ) {
    await model.resetHighlight();
  }

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer(
      false,
    );
  } else {
    await fragmentManager.core.update(
      true,
    );
  }

  isFlowing.value =
    false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value =
    true;

  flowMessage.value =
    "Realce das válvulas removido.";
}

function isValveElementType(
  elementType: MepElementType,
) {
  return (
    elementType === "isolationValve" ||
    elementType === "normallyOpenValve" ||
    elementType === "normallyClosedValve"
  );
}

function isConfirmedShutoffValve(
  element?: MepElement,
) {
  if (
    !element ||
    !isValveElementType(
      element.elementType,
    ) ||
    element.isShutoffValve !== true
  ) {
    return false;
  }

  const key = elementKey(
    element.modelId,
    element.localId,
  );

  return !excludedValveKeys.has(
    key,
  );
}

function getNormalValveState(
  elementType: MepElementType,
) {
  if (elementType === "normallyClosedValve") {
    return "closed";
  }

  return "open";
}

function getValveTypeFromIfcState(
  stateValue?: string | null,
): MepElementType {
  const normalizedState = String(
    stateValue ?? "",
  )
    .trim()
    .toLowerCase()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .replace(/[\s._-]/g, "");

  const normallyClosedValues = [
    "nc",
    "nf",
    "normallyclosed",
    "normalmentefechada",
    "normalmentefechado",
  ];

  if (
    normallyClosedValues.includes(
      normalizedState,
    )
  ) {
    return "normallyClosedValve";
  }

  return "normallyOpenValve";
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

async function highlightValveFromDropdown(
  node: FlowNode,
) {
  isValveFocusModeActive.value =
    true;

  flowPreparationRunId++;

  isPreparingFlowAnimation.value =
    false;

  isFlowAnimationReady.value =
    false;

  flowPreparationProgress.value =
    0;

  hasFlowPreparationError.value =
    false;

  isFlowing.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isManualFlowAnimationRunning.value =
    false;

  isFlowManuallyPaused.value =
    true;

  highlightedSavedRouteId.value =
    null;

  await clearFlowVisualsForRouteHighlight();

  await new Promise<void>(
  (resolve) => {
    requestAnimationFrame(
      () => resolve(),
    );
  },
);

await clearPersistentCircuitHighlights();

for (
  const loadedModel of
    loadedModels.values()
) {
  await loadedModel.resetHighlight();
}

await fragmentManager.core.update(
  true,
);

  const model =
    loadedModels.get(
      node.modelId,
    );

  if (!model) {
    flowMessage.value =
      "Não foi possível encontrar o modelo da válvula.";

    return;
  }

  await model.highlight(
    [node.localId],
    createHighlight(
      0xff00ff,
      "valve-dropdown-selected",
    ),
  );

  highlightedValveFromDropdown.value = {
    modelId:
      node.modelId,

    localId:
      node.localId,
  };

  await fragmentManager.core.update(
    true,
  );
}

async function findClosestProtectedRouteMatch(
  valveNode: FlowNode,
  protectedRoutes: SavedRoute[],
): Promise<AutomaticValveRouteMatch | null> {
  const valveCenter =
    await getNodeCenter(
      valveNode,
    );

  if (!valveCenter) {
    return null;
  }

  const loadedModelIds = [
    ...loadedModels.keys(),
  ];

  const fallbackModelId =
    loadedModelIds[0];

  if (!fallbackModelId) {
    return null;
  }

  const matches:
    Array<{
      route: SavedRoute;
      closestNode: FlowNode;
      distance: number;
    }> = [];

  for (const route of protectedRoutes) {
    let closestRouteNode:
      FlowNode | null = null;

    let closestRouteDistance =
      Number.POSITIVE_INFINITY;

    for (const routeNode of route.path) {
      const adaptedNode: FlowNode =
        loadedModels.has(
          routeNode.modelId,
        )
          ? {
              modelId:
                routeNode.modelId,
              localId:
                routeNode.localId,
            }
          : {
              modelId:
                fallbackModelId,
              localId:
                routeNode.localId,
            };

      const nodeCenter =
        await getNodeCenter(
          adaptedNode,
        );

      if (!nodeCenter) {
        continue;
      }

      const distance =
        valveCenter.distanceTo(
          nodeCenter,
        );

      if (
        distance <
        closestRouteDistance
      ) {
        closestRouteDistance =
          distance;

        closestRouteNode =
          adaptedNode;
      }
    }

    if (
      closestRouteNode &&
      Number.isFinite(
        closestRouteDistance,
      )
    ) {
      matches.push({
        route,
        closestNode:
          closestRouteNode,
        distance:
          closestRouteDistance,
      });
    }
  }

  matches.sort(
    (firstMatch, secondMatch) =>
      firstMatch.distance -
      secondMatch.distance,
  );

  const closestMatch =
    matches[0];

  if (!closestMatch) {
    return null;
  }

  const alternativeMatch =
    matches[1];

  const ambiguityTolerance =
    Math.max(
      0.05,
      closestMatch.distance * 0.15,
    );

  const ambiguous =
    !!alternativeMatch &&
    (
      alternativeMatch.distance -
      closestMatch.distance
    ) <= ambiguityTolerance;

  return {
    route:
      closestMatch.route,

    closestNode:
      closestMatch.closestNode,

    distance:
      closestMatch.distance,

    ambiguous,

    alternativeRouteName:
      ambiguous
        ? alternativeMatch?.route.name
        : undefined,

    alternativeDistance:
      ambiguous
        ? alternativeMatch?.distance
        : undefined,
  };
}

function getProtectedRouteDownstreamPipes(
  route: SavedRoute,
  startNode: FlowNode,
) {
  const loadedModelIds = [
    ...loadedModels.keys(),
  ];

  const fallbackModelId =
    loadedModelIds[0];

  if (!fallbackModelId) {
    return [];
  }

  const adaptedPath =
    route.path.map(
      (node): FlowNode => {
        if (
          loadedModels.has(
            node.modelId,
          )
        ) {
          return {
            modelId:
              node.modelId,
            localId:
              node.localId,
          };
        }

        return {
          modelId:
            fallbackModelId,
          localId:
            node.localId,
        };
      },
    );

  const startIndex =
    adaptedPath.findIndex(
      (node) =>
        isSameNode(
          node,
          startNode,
        ),
    );

  if (startIndex === -1) {
    return [];
  }

  return adaptedPath
    .slice(startIndex)
    .map(
      (
        node,
      ): ValveControlledPipeLink => ({
        modelId:
          node.modelId,

        localId:
          node.localId,

        temperature:
          route.temperature,

        routeId:
          route.id,

        switchMode: "none",
      }),
    );
}

function getAutomaticValvePreviewStatusLabel(
  status:
    AutomaticValveAssociationPreview["status"],
) {
  if (status === "safe") {
    return "Pronta para confirmar";
  }

  if (status === "noMatch") {
    return "Sem percurso protegido próximo";
  }

  if (status === "noDownstream") {
    return "Sem tubos a jusante";
  }

  return "Associação ambígua";
}

function toggleAutomaticValvePreviewAcceptance(
  preview:
    AutomaticValveAssociationPreview,
) {
  if (preview.status !== "safe") {
    return;
  }

  preview.accepted =
    !preview.accepted;
}

function selectAllSafeValveAssociationPreviews() {
  for (
    const preview of
      automaticValveAssociationPreviews
  ) {
    preview.accepted =
      preview.status === "safe";
  }
}

function clearAutomaticValveAssociationPreviewSelection() {
  for (
    const preview of
      automaticValveAssociationPreviews
  ) {
    preview.accepted = false;
  }
}

function getAcceptedAutomaticValvePreviewCount() {
  return automaticValveAssociationPreviews.filter(
    (preview) =>
      preview.status === "safe" &&
      preview.accepted,
  ).length;
}

async function clearAutomaticValveAssociationPreviewHighlight() {
  for (const model of loadedModels.values()) {
    await model.resetHighlight();
  }

  await fragmentManager.core.update(
    true,
  );

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer(
  false,
);
  }
}

async function highlightAutomaticValveAssociationPreview(
  preview: AutomaticValveAssociationPreview,
) {
  if (
    !preview.routeId ||
    !preview.closestNode
  ) {
    flowMessage.value =
      "Esta proposta não tem um percurso válido para realçar.";

    return;
  }

  await clearAutomaticValveAssociationPreviewHighlight();

  const route =
    savedRoutes.find(
      (savedRoute) =>
        savedRoute.id ===
        preview.routeId,
    );

  if (!route) {
    flowMessage.value =
      "Não foi possível encontrar o percurso proposto.";

    return;
  }

  const downstreamPipes =
    getProtectedRouteDownstreamPipes(
      route,
      preview.closestNode,
    );

  if (!downstreamPipes.length) {
    flowMessage.value =
      "Esta proposta não tem tubos a jusante para realçar.";

    return;
  }

  await clearManagedValveHighlight();

  const idsByModel =
    new Map<string, number[]>();

  for (const pipeNode of downstreamPipes) {
    const modelIds =
      idsByModel.get(
        pipeNode.modelId,
      ) ?? [];

    modelIds.push(
      pipeNode.localId,
    );

    idsByModel.set(
      pipeNode.modelId,
      modelIds,
    );
  }

  for (
    const [modelId, localIds] of
      idsByModel
  ) {
    const model =
      loadedModels.get(modelId);

    if (
      !model ||
      localIds.length === 0
    ) {
      continue;
    }

    await model.highlight(
      localIds,
      createHighlight(
        0x00e5ff,
        "automatic-valve-preview-pipes",
      ),
    );
  }

  const valveModel =
    loadedModels.get(
      preview.valveNode.modelId,
    );

  if (valveModel) {
    await valveModel.highlight(
      [
        preview.valveNode.localId,
      ],
      createHighlight(
        0xff00ff,
        "automatic-valve-preview-valve",
      ),
    );
  }

  await fragmentManager.core.update(
    true,
  );

  flowMessage.value =
    'Proposta da válvula "' +
    preview.valveLabel +
    '" realçada: ' +
    downstreamPipes.length +
    " tubo(s) no percurso " +
    '"' +
    route.name +
    '".';
}

async function closeAutomaticValveAssociationPreview() {
  await clearAutomaticValveAssociationPreviewHighlight();

  isAutomaticValveAssociationPreviewOpen.value =
    false;

  automaticValveAssociationPreviews.splice(
    0,
  );

  flowMessage.value =
    "Revisão das associações automáticas fechada.";
}

async function confirmAutomaticValveAssociationPreviews() {
  const acceptedPreviews =
    automaticValveAssociationPreviews.filter(
      (preview) =>
        preview.status === "safe" &&
        preview.accepted,
    );

  if (!acceptedPreviews.length) {
    flowMessage.value =
      "Seleciona pelo menos uma proposta segura.";

    return;
  }

  const shouldConfirm =
    window.confirm(
      "Serão guardadas " +
        acceptedPreviews.length +
        " associação(ões) automática(s).\n\n" +
        "Queres continuar?",
    );

  if (!shouldConfirm) {
    flowMessage.value =
      "Confirmação das associações cancelada.";

    return;
  }

  let associatedCount = 0;
  let failedCount = 0;
  let closedValveCount = 0;

  for (
    const preview of
      acceptedPreviews
  ) {
    if (
      !preview.routeId ||
      !preview.closestNode
    ) {
      failedCount++;

      continue;
    }

    const route =
      savedRoutes.find(
        (savedRoute) =>
          savedRoute.id ===
          preview.routeId,
      );

    if (!route) {
      failedCount++;

      continue;
    }

    const downstreamPipes =
      getProtectedRouteDownstreamPipes(
        route,
        preview.closestNode,
      );

    if (!downstreamPipes.length) {
      failedCount++;

      continue;
    }

    const valveElement =
      mepElements[
        elementKey(
          preview.valveNode.modelId,
          preview.valveNode.localId,
        )
      ];

    if (!valveElement) {
      failedCount++;

      continue;
    }

const linkedPipes =
  downstreamPipes.map(
    (
      pipeNode,
    ): ValveControlledPipeLink => ({
      ...pipeNode,

      switchMode: "none",
    }),
  );

    valveControlledPipeLinks.set(
      preview.valveKey,
      linkedPipes,
    );

    valveBlockedPipeLinks.delete(
      preview.valveKey,
    );

    if (
      valveElement.state ===
      "closed"
    ) {
      for (
        const linkedPipe of
          linkedPipes
      ) {
        blockPipeForRoute(
          linkedPipe.routeId,
          linkedPipe,
        );
      }

      valveBlockedPipeLinks.set(
        preview.valveKey,
        linkedPipes,
      );

      closedValveCount++;
    }

    associatedCount++;
  }

  saveValvePipeLinksToStorage();
  saveMepElementsToStorage();

  updateBlockedCount();
  updateManualStats();

  await clearAutomaticValveAssociationPreviewHighlight();

  isAutomaticValveAssociationPreviewOpen.value =
    false;

  automaticValveAssociationPreviews.splice(
    0,
  );

  if (
    associatedCount > 0 &&
    (
      countAssignments() > 0 ||
      flowConnections.length > 0
    )
  ) {
    await rebuildManualFlowLayer(
  false,
);
  }

  const resultMessage =
    "Associações automáticas guardadas.\n\n" +
    "Válvulas associadas: " +
    associatedCount +
    "\n" +
    "Válvulas fechadas com bloqueio aplicado: " +
    closedValveCount +
    "\n" +
    "Propostas que não foi possível guardar: " +
    failedCount;

  window.alert(
    resultMessage,
  );

  flowMessage.value =
    associatedCount +
    " válvula(s) associada(s) automaticamente.";
}

async function handleAutomaticValveAssociationClick() {
  await automaticallyLinkAllValvesByDirection();
}

async function automaticallyLinkAllValvesByDirection() {
  if (
    isPreparingAutomaticValveAssociations.value
  ) {
    return;
  }

  const detectedValves =
    Object.values(
      mepElements,
    ).filter(
      (element) =>
        isConfirmedShutoffValve(
          element,
        ),
    );

  if (!detectedValves.length) {
    window.alert(
      "Não existem válvulas de corte disponíveis.\n\n" +
        "Confirma se o scan reconheceu válvulas NA ou NF.",
    );

    flowMessage.value =
      "Não existem válvulas de corte disponíveis.";

    return;
  }

  const validatedRoutes =
    savedRoutes.filter(
      (route) =>
        !route.hidden &&
        route.locked &&
        route.path.length > 0,
    );

  if (!validatedRoutes.length) {
    window.alert(
      "Nenhum percurso visível está protegido.\n\n" +
        "Revê os sentidos das setas e protege " +
        "os percursos que consideras validados.",
    );

    flowMessage.value =
      "Nenhum percurso visível está protegido.";

    return;
  }

  const unlinkedValves =
    detectedValves.filter(
      (element) => {
        const valveNode: FlowNode = {
          modelId:
            element.modelId,

          localId:
            element.localId,
        };

        return (
          getLinkedPipesForValveNode(
            valveNode,
          ).length === 0
        );
      },
    );

  if (!unlinkedValves.length) {
    window.alert(
      "Todas as válvulas de corte já têm uma associação guardada.",
    );

    flowMessage.value =
      "Todas as válvulas de corte já estão associadas.";

    return;
  }

  isPreparingAutomaticValveAssociations.value =
    true;

  automaticValveAssociationPreviews.splice(
    0,
  );

  automaticValveAssociationProgress.value =
    0;

  automaticValveAssociationTotal.value =
    unlinkedValves.length;

  flowMessage.value =
    "A calcular propostas de associação automática...";

  try {
    for (
      let valveIndex = 0;
      valveIndex <
      unlinkedValves.length;
      valveIndex++
    ) {
      const valve =
        unlinkedValves[valveIndex];

      automaticValveAssociationProgress.value =
        valveIndex + 1;

      flowMessage.value =
        "A calcular associação da válvula " +
        (valveIndex + 1) +
        " de " +
        unlinkedValves.length +
        "...";

      await new Promise<void>(
        (resolve) => {
          requestAnimationFrame(
            () => resolve(),
          );
        },
      );

      const valveNode: FlowNode = {
        modelId:
          valve.modelId,

        localId:
          valve.localId,
      };

      const valveKey =
        nodeKey(
          valveNode,
        );

      const valveLabel =
        valve.name ||
        valve.valveIdentificationText ||
        "Válvula #" +
          valve.localId;

      const normalStateLabel =
        valve.elementType ===
        "normallyClosedValve"
          ? "NF"
          : "NA";

      const currentStateLabel =
        valve.state === "closed"
          ? "Fechada"
          : "Aberta";

      const closestMatch =
        await findClosestProtectedRouteMatch(
          valveNode,
          validatedRoutes,
        );

      if (!closestMatch) {
        automaticValveAssociationPreviews.push({
          valveKey,
          valveNode,
          valveLabel,
          normalStateLabel,
          currentStateLabel,

          routeId: null,

          routeName:
            "Nenhum percurso encontrado",

          closestNode: null,

          distance: null,

          downstreamPipeCount: 0,

          status: "noMatch",

          accepted: false,
        });

        continue;
      }

      if (closestMatch.ambiguous) {
        automaticValveAssociationPreviews.push({
          valveKey,
          valveNode,
          valveLabel,
          normalStateLabel,
          currentStateLabel,

          routeId:
            closestMatch.route.id,

          routeName:
            closestMatch.route.name +
            " ou " +
            (
              closestMatch
                .alternativeRouteName ??
              "outro percurso"
            ),

          closestNode:
            closestMatch.closestNode,

          distance:
            closestMatch.distance,

          downstreamPipeCount: 0,

          status: "ambiguous",

          accepted: false,
        });

        continue;
      }

      const downstreamPipes =
        getProtectedRouteDownstreamPipes(
          closestMatch.route,
          closestMatch.closestNode,
        );

      if (!downstreamPipes.length) {
        automaticValveAssociationPreviews.push({
          valveKey,
          valveNode,
          valveLabel,
          normalStateLabel,
          currentStateLabel,

          routeId:
            closestMatch.route.id,

          routeName:
            closestMatch.route.name,

          closestNode:
            closestMatch.closestNode,

          distance:
            closestMatch.distance,

          downstreamPipeCount: 0,

          status: "noDownstream",

          accepted: false,
        });

        continue;
      }

      automaticValveAssociationPreviews.push({
        valveKey,
        valveNode,
        valveLabel,
        normalStateLabel,
        currentStateLabel,

        routeId:
          closestMatch.route.id,

        routeName:
          closestMatch.route.name,

        closestNode:
          closestMatch.closestNode,

        distance:
          closestMatch.distance,

        downstreamPipeCount:
          downstreamPipes.length,

        status: "safe",

        accepted: true,
      });
    }

    isAutomaticValveAssociationPreviewOpen.value =
      true;

    const safeCount =
      automaticValveAssociationPreviews.filter(
        (preview) =>
          preview.status === "safe",
      ).length;

    const reviewCount =
      automaticValveAssociationPreviews.length -
      safeCount;

    flowMessage.value =
      safeCount +
      " associação(ões) pronta(s) para revisão. " +
      reviewCount +
      " caso(s) precisam de verificação.";
  } catch (error) {
    console.error(
      "Erro ao preparar associações automáticas:",
      error,
    );

    window.alert(
      "Não foi possível preparar as associações automáticas.\n\n" +
        (
          error instanceof Error
            ? error.message
            : String(error)
        ),
    );

    flowMessage.value =
      "Não foi possível preparar as associações automáticas.";
  } finally {
    isPreparingAutomaticValveAssociations.value =
      false;

    automaticValveAssociationProgress.value =
      0;

    automaticValveAssociationTotal.value =
      0;
  }
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

if (
  countAssignments() > 0 ||
  flowConnections.length > 0
) {
  await rebuildManualFlowLayer(
    false,
  );
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

  await rebuildManualFlowLayer(
  false,
);

  flowMessage.value =
    "Associação da válvula removida.";
}

async function clearManualValveControlledElementColors(
  controlledElements:
    ManualValveControlledPipe[],
) {
  const idsByModel =
    new Map<string, Set<number>>();

  for (
    const controlledElement of
      controlledElements
  ) {
    const modelIds =
      idsByModel.get(
        controlledElement.modelId,
      ) ??
      new Set<number>();

    modelIds.add(
      controlledElement.localId,
    );

    idsByModel.set(
      controlledElement.modelId,
      modelIds,
    );
  }

  for (
    const [modelId, localIds] of
      idsByModel
  ) {
    const model =
      loadedModels.get(modelId);

    if (
      !model ||
      localIds.size === 0
    ) {
      continue;
    }

    await model.resetHighlight(
      [...localIds],
    );
  }

  await fragmentManager.core.update(
    true,
  );
}

async function setSelectedValvesState(
  state: "open" | "closed",
) {
  const valveNodes =
    getValveNodesForControl();

  if (!valveNodes.length) {
    flowMessage.value =
      "Seleciona primeiro uma válvula.";

    return;
  }

  let changedCount =
    0;

  const affectedControlledElements =
  new Map<
    string,
    ManualValveControlledPipe
  >();

  for (
    const valveNode of valveNodes
  ) {
    const {
      modelId,
      localId,
    } = valveNode;

    if (
      !isIsolationValve(
        modelId,
        localId,
      )
    ) {
      continue;
    }

    const valveKey =
      elementKey(
        modelId,
        localId,
      );

    const existingElement =
      mepElements[valveKey];

    if (
      !existingElement ||
      !isValveElementType(
        existingElement.elementType,
      )
    ) {
      continue;
    }

    if (
      existingElement.state ===
      state
    ) {
      continue;
    }

    mepElements[valveKey] = {
      ...existingElement,
      modelId,
      localId,
      state,
    };

    const controlledPipes =
      manualValveControlledPipes.get(
        valveKey,
      ) ?? [];

    for (
  const controlledElement of
    controlledPipes
) {
  const controlledElementKey =
    nodeKey(
      controlledElement,
    );

  affectedControlledElements.set(
    controlledElementKey,
    {
      modelId:
        controlledElement.modelId,

      localId:
        controlledElement.localId,
    },
  );
}

    changedCount++;
  }

  if (!changedCount) {
    flowMessage.value =
      state === "closed"
        ? "A válvula selecionada já está fechada."
        : "A válvula selecionada já está aberta.";

    return;
  }

  saveMepElementsToStorage();

  isValveFocusModeActive.value =
    false;

  if (
    highlightedValveFromDropdown.value
  ) {
    const highlightedValveModel =
      loadedModels.get(
        highlightedValveFromDropdown.value
          .modelId,
      );

    if (highlightedValveModel) {
      await highlightedValveModel
        .resetHighlight([
          highlightedValveFromDropdown.value
            .localId,
        ]);
    }

    highlightedValveFromDropdown.value =
      null;
  }

  isFlowing.value =
    false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value =
    true;

  if (
  state === "closed" &&
  affectedControlledElements.size > 0
) {
  await clearManualValveControlledElementColors(
    [
      ...affectedControlledElements
        .values(),
    ],
  );
}

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer(
      false,
    );
  } else {
    await fragmentManager.core.update(
      true,
    );
  }

  await showSelectedMepElementInfo();

  flowMessage.value =
  state === "closed"
    ? changedCount +
      " válvula(s) fechada(s). " +
      affectedControlledElements.size +
      " elemento(s) controlado(s) ficaram sem cor e sem setas."
    : changedCount +
      " válvula(s) aberta(s). " +
      "As cores e as setas dos elementos que já não estão bloqueados foram repostas.";
}

function getAdaptedSavedRouteNodes(
  route: SavedRoute,
) {
  const loadedModelIds = [
    ...loadedModels.keys(),
  ];

  const fallbackModelId =
    loadedModelIds[0];

  if (!fallbackModelId) {
    return [];
  }

  return route.path.map(
    (node): FlowNode => {
      if (
        loadedModels.has(
          node.modelId,
        )
      ) {
        return {
          modelId: node.modelId,
          localId: node.localId,
        };
      }

      return {
        modelId: fallbackModelId,
        localId: node.localId,
      };
    },
  );
}

function areSelectedFlowArrowsHidden() {
  if (!selectedCount.value) {
    return false;
  }

  for (
    const [modelId, localIds] of
      selectedItems
  ) {
    for (const localId of localIds) {
      if (
        !isFlowArrowHidden(
          modelId,
          localId,
        )
      ) {
        return false;
      }
    }
  }

  return true;
}

async function toggleSelectedFlowArrows() {
  if (!selectedCount.value) {
    flowMessage.value =
      "Seleciona primeiro um ou mais tubos no modelo.";

    return;
  }

  if (
    areSelectedFlowArrowsHidden()
  ) {
    await showSelectedFlowArrows();

    return;
  }

  await hideSelectedFlowArrows();
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

async function syncSelectedPipesForSavedRoute(
  route: SavedRoute,
) {
  if (route.locked) {
    flowMessage.value =
      'O percurso "' +
      route.name +
      '" está protegido. Desprotege primeiro.';

    return;
  }

  if (!selectedCount.value) {
    flowMessage.value =
      "Seleciona primeiro um ou mais tubos desse percurso.";

    return;
  }

  let changedCount = 0;
  let ignoredCount = 0;

  for (
    const [modelId, localIds] of
      selectedItems
  ) {
    const syncedSet =
      getSyncedDirectionSet(
        modelId,
      );

    for (const localId of localIds) {
      const node: FlowNode = {
        modelId,
        localId,
      };

      if (
        !routeContainsAdaptedNode(
          route,
          node,
        )
      ) {
        ignoredCount++;

        continue;
      }

      if (syncedSet.has(localId)) {
        syncedSet.delete(localId);
      } else {
        syncedSet.add(localId);
      }

      toggleReversedPipeDirection(
        modelId,
        localId,
      );

      changedCount++;
    }

    if (syncedSet.size === 0) {
      syncedPipeDirections.delete(
        modelId,
      );
    }
  }

  if (!changedCount) {
    flowMessage.value =
      ignoredCount > 0
        ? "Os tubos selecionados não pertencem a este percurso."
        : "Nenhum tubo foi sincronizado.";

    return;
  }

  saveSyncedPipeDirectionsToStorage();
  saveReversedDirectionsToStorage();

  selectedRouteIdsForSimulation.add(
    route.id,
  );

  isFlowing.value = false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value = false;

  await rebuildManualFlowLayer();

  if (
    !hasFlowPreparationError.value &&
    isFlowAnimationReady.value &&
    pipeParticles.length > 0
  ) {
    isFlowing.value = true;

    isManualFlowAnimationRunning.value =
      true;

    isCentralSimulationRunning.value =
      false;

    isFlowManuallyPaused.value = false;
  }

  flowMessage.value =
    changedCount +
    " tubo(s) sincronizado(s) no percurso " +
    '"' +
    route.name +
    '".' +
    (
      ignoredCount > 0
        ? " " +
          ignoredCount +
          " elemento(s) ignorado(s) por não pertencerem ao percurso."
        : ""
    );
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

async function clearBlockedPipes() {
  blockedPipes.clear();
  blockedRoutePipes.clear();
  valveBlockedPipeLinks.clear();

  if (
    !flowConnections.length &&
    savedRoutes.length &&
    loadedModels.size
  ) {
    await applyAllSavedRoutes();
  }

  for (
    const key of
      Object.keys(mepElements)
  ) {
    const element =
      mepElements[key];

    if (
      !isValveElementType(
        element.elementType,
      )
    ) {
      continue;
    }

    const valveNode: FlowNode = {
      modelId:
        element.modelId,

      localId:
        element.localId,
    };

    const valveKey =
      nodeKey(
        valveNode,
      );

    const linkedPipes =
      valveControlledPipeLinks.get(
        valveKey,
      ) ?? [];

    restoreValveLinkedPipesToOriginalCircuit(
      linkedPipes,
    );

    if (element.state === "closed") {
      for (
        const pipeNode of
          linkedPipes
      ) {
        blockPipeForRoute(
          pipeNode.routeId,
          pipeNode,
        );
      }

      valveBlockedPipeLinks.set(
        valveKey,
        linkedPipes,
      );
    }
  }

  updateBlockedCount();

  await rebuildManualFlowLayer(
    false,
  );

  saveMepElementsToStorage();

  flowMessage.value =
    "Estados guardados das válvulas aplicados.";
}

async function updateAutomaticDirectionNeighbors(
  circuit: PipeCircuit,
  orderedNodes: Array<
    FlowNode & {
      revitElementId: number;
    }
  >,
) {
  const nodesByModel = new Map<
    string,
    Array<
      FlowNode & {
        revitElementId: number;
      }
    >
  >();

  for (const node of orderedNodes) {
    const modelNodes =
      nodesByModel.get(node.modelId) ?? [];

    modelNodes.push(node);
    nodesByModel.set(node.modelId, modelNodes);
  }

  for (const [modelId, modelNodes] of nodesByModel) {
    const model = loadedModels.get(modelId);

    if (!model || !modelNodes.length) {
      continue;
    }

    const localIds = modelNodes.map(
      (node) => node.localId,
    );

    const boxes = await model.getBoxes(localIds);

    const graphItems = new Map<
      number,
      PipeGraphItem
    >();

    const nodesByLocalId = new Map<
      number,
      FlowNode & {
        revitElementId: number;
      }
    >();

    for (
      let index = 0;
      index < modelNodes.length;
      index++
    ) {
      const node = modelNodes[index];

      const graphItem = graphItemFromBox(
        modelId,
        node.localId,
        boxes[index],
      );

      if (!graphItem) {
        continue;
      }

      const geometryAxis =
        await getPipeGeometryAxis({
          modelId,
          localId: node.localId,
        });

      if (geometryAxis) {
        graphItem.center.copy(
          geometryAxis.center,
        );

        graphItem.endpoints = [
          geometryAxis.start.clone(),
          geometryAxis.end.clone(),
        ];
      }

      graphItems.set(
        node.localId,
        graphItem,
      );

      nodesByLocalId.set(
        node.localId,
        node,
      );
    }

    const adjacency = new Map<number, number[]>();

    for (const node of modelNodes) {
      adjacency.set(node.localId, []);
    }

    for (
      let firstIndex = 0;
      firstIndex < modelNodes.length;
      firstIndex++
    ) {
      const firstNode = modelNodes[firstIndex];
      const firstItem =
        graphItems.get(firstNode.localId);

      if (!firstItem) {
        continue;
      }

      for (
        let secondIndex = firstIndex + 1;
        secondIndex < modelNodes.length;
        secondIndex++
      ) {
        const secondNode = modelNodes[secondIndex];
        const secondItem =
          graphItems.get(secondNode.localId);

        if (!secondItem) {
          continue;
        }

        const distance = pipeConnectionDistance(
          firstItem,
          secondItem,
        );

        const tolerance = pipeConnectionTolerance(
          firstItem,
          secondItem,
        );

        if (distance > tolerance) {
          continue;
        }

        adjacency
          .get(firstNode.localId)
          ?.push(secondNode.localId);

        adjacency
          .get(secondNode.localId)
          ?.push(firstNode.localId);
      }
    }

    const unvisited = new Set(
      nodesByLocalId.keys(),
    );

    while (unvisited.size) {
            const unvisitedLocalIds = [
        ...unvisited,
      ];

      const endpointLocalId =
        unvisitedLocalIds.find(
          (localId) => {
            const connectedIds =
              adjacency.get(localId) ?? [];

            const connectedUnvisitedCount =
              connectedIds.filter(
                (connectedLocalId) =>
                  unvisited.has(
                    connectedLocalId,
                  ),
              ).length;

            return (
              connectedUnvisitedCount <= 1
            );
          },
        );

      const componentStartLocalId =
        endpointLocalId ??
        unvisitedLocalIds[0];

      const componentStart =
        componentStartLocalId !== undefined
          ? nodesByLocalId.get(
              componentStartLocalId,
            )
          : undefined;

      if (!componentStart) {
        break;
      }

      const queue: number[] = [
        componentStart.localId,
      ];

      const parentByLocalId =
        new Map<number, number | null>();

      parentByLocalId.set(
        componentStart.localId,
        null,
      );

      unvisited.delete(
        componentStart.localId,
      );

      while (queue.length) {
        const currentLocalId = queue.shift();

        if (currentLocalId === undefined) {
          continue;
        }

        const currentNode =
          nodesByLocalId.get(currentLocalId);

        if (!currentNode) {
          continue;
        }

        const connectedUnvisitedNodes = (
          adjacency.get(currentLocalId) ?? []
        )
          .filter((localId) =>
            unvisited.has(localId),
          )
          .map((localId) =>
            nodesByLocalId.get(localId),
          )
          .filter(
            (
              node,
            ): node is FlowNode & {
              revitElementId: number;
            } => !!node,
          );
        
        for (
          const nextNode of
            connectedUnvisitedNodes
        ) {
          parentByLocalId.set(
            nextNode.localId,
            currentLocalId,
          );

          unvisited.delete(nextNode.localId);
          queue.push(nextNode.localId);
        }

        const parentLocalId =
          parentByLocalId.get(
            currentLocalId,
          ) ?? null;

        const firstChild =
          connectedUnvisitedNodes[0] ?? null;

        automaticDirectionNeighbors.set(
          automaticDirectionKey(
            circuit,
            currentNode,
          ),
          {
            previous:
              parentLocalId !== null
                ? {
                    modelId,
                    localId: parentLocalId,
                  }
                : null,

            next: firstChild
              ? {
                  modelId,
                  localId:
                    firstChild.localId,
                }
              : null,
          },
        );
      }
    }
  }
}

async function getPipeDirectionHints(
  node: FlowNode,
): Promise<PipeDirectionHints> {
  const circuit =
    getNodeTemperature(node);

  if (!circuit) {
    return {};
  }

  const automaticNeighbors =
    automaticDirectionNeighbors.get(
      automaticDirectionKey(
        circuit,
        node,
      ),
    );

  if (automaticNeighbors) {
    const hints: PipeDirectionHints = {};

    if (automaticNeighbors.previous) {
      hints.upstream =
        await getNodeCenter(
          automaticNeighbors.previous,
        );
    }

    if (automaticNeighbors.next) {
      hints.downstream =
        await getNodeCenter(
          automaticNeighbors.next,
        );
    }

    if (
      hints.upstream ||
      hints.downstream
    ) {
      return hints;
    }
  }

  const hints: PipeDirectionHints = {};

  for (const connection of flowConnections) {
    if (
      connection.temperature !== circuit
    ) {
      continue;
    }

    if (isSameNode(connection.to, node)) {
      hints.upstream =
        await getNodeCenter(
          connection.from,
        );
    }

    if (isSameNode(connection.from, node)) {
      hints.downstream =
        await getNodeCenter(
          connection.to,
        );
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
  if (
  circuit ===
  UNASSIGNED_ROUTE_CIRCUIT
) {
  return 0;
}

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
  if (
  circuit ===
  UNASSIGNED_ROUTE_CIRCUIT
) {
  return "sem ciclo";
}

  const definition = getCycleCircuitDefinition(circuit);

  if (definition) {
  if (
    definition.cycleNumber === 0
  ) {
    return (
      definition.name +
      " · sem ciclo"
    );
  }

  return (
    definition.name +
    " " +
    getCycleDisplayName(
      definition.cycleNumber,
    )
  );
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
  setActiveIfcStorageItem(
    CYCLE_CIRCUITS_STORAGE_KEY,
    JSON.stringify(
      cycleCircuitDefinitions,
    ),
  );
}

function loadCycleCircuitDefinitionsFromStorage() {
  const saved =
    getActiveIfcStorageItem(
      CYCLE_CIRCUITS_STORAGE_KEY,
    );

  if (saved) {
    try {
      const parsed =
        JSON.parse(
          saved,
        ) as CycleCircuitDefinition[];

      cycleCircuitDefinitions.splice(
        0,
      );

      cycleCircuitDefinitions.push(
        ...parsed,
      );
    } catch (error) {
      console.error(
        "Erro ao carregar circuitos dos ciclos:",
        error,
      );
    }
  }

  ensureCycleCircuitDefinitions();
}

function normalizeSystemMatchText(
  value?: string,
) {
  return String(value ?? "")
    .trim()
    .toLowerCase()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .replace(/[\s._-]/g, "");
}

function getCircuitForValveSystem(
  valve: MepElement,
) {
  const normalizedValveSystemName =
    normalizeSystemMatchText(
      valve.systemName,
    );

  if (!normalizedValveSystemName) {
    return null;
  }

  return (
    cycleCircuitDefinitions.find(
      (circuit) =>
        normalizeSystemMatchText(
          circuit.name,
        ) ===
        normalizedValveSystemName,
    ) ?? null
  );
}

function getRoutesForValveSystem(
  valve: MepElement,
) {
  const circuit =
    getCircuitForValveSystem(
      valve,
    );

  if (!circuit) {
    return [] as SavedRoute[];
  }

  return savedRoutes.filter(
    (route) =>
      route.temperature ===
      circuit.key,
  );
}

function getSingleAutomaticRouteForValve(
  valve: MepElement,
) {
  const compatibleRoutes =
    getRoutesForValveSystem(
      valve,
    );

  if (compatibleRoutes.length !== 1) {
    return null;
  }

  return compatibleRoutes[0];
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
    const circuits =
  getCycleCircuitDefinitionsForCycle(
    cycleNumber,
  ).filter(
    (circuit) =>
      savedRoutes.some(
        (route) =>
          route.temperature ===
          circuit.key,
      ),
  );

    if (!circuits.length) {
      continue;
    }

    groups.push({
  cycleNumber,

  cycleName:
    getCycleLegendDisplayName(
      cycleNumber,
    ),

  circuits:
    [...circuits]
      .map((circuit) => {
        const representativeRoute =
          savedRoutes.find(
            (route) =>
              route.temperature ===
              circuit.key &&
              !route.hidden,
          ) ??
          savedRoutes.find(
            (route) =>
              route.temperature ===
              circuit.key,
          );

        return {
          ...circuit,

          color:
            representativeRoute
              ? getSavedRouteGroupColor(
                  representativeRoute,
                )
              : circuit.color,
        };
      })
      .sort(
        (
          firstCircuit,
          secondCircuit,
        ) =>
          firstCircuit.name.localeCompare(
            secondCircuit.name,
            "pt-PT",
            {
              sensitivity: "base",
              numeric: true,
            },
          ),
      ),
});
  }

  return groups;
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

function getNewRouteCycleNumber() {
  const selectedCycleNumber =
    Number(
      newRouteCycleSelection.value,
    );

  if (
    !Number.isFinite(
      selectedCycleNumber,
    ) ||
    selectedCycleNumber < 1
  ) {
    return 0;
  }

  return selectedCycleNumber;
}

function getNewRouteCircuitDisplayName() {
  const typedName =
    newRouteCircuitName.value.trim();

  if (typedName) {
    return typedName;
  }

  return getDefaultNewRouteCircuitName(
    newRouteCircuitKind.value,
  );
}

function createNewRouteCircuitDefinition() {
  const circuitKey =
    "new-route-circuit-" +
    crypto.randomUUID();

  const circuitName =
    getNewRouteCircuitDisplayName();

  const circuitColor =
    newRouteCircuitColor.value ||
    getDefaultNewRouteCircuitColor(
      newRouteCircuitKind.value,
    );

  const circuit:
    CycleCircuitDefinition = {
      key:
        circuitKey,

      cycleNumber:
        getNewRouteCycleNumber(),

      kind:
        newRouteCircuitKind.value,

      name:
        circuitName,

      color:
        circuitColor,

      defaultColor:
        circuitColor,

      locked:
        false,
    };

  cycleCircuitDefinitions.push(
    circuit,
  );

  manualAssignments[circuitKey] =
    new Map();

  newRouteDraftCircuitKey.value =
    circuitKey;

  return circuitKey;
}

function removeUnusedNewRouteCircuit(
  circuitKey: string,
) {
  const circuitIndex =
    cycleCircuitDefinitions.findIndex(
      (circuit) =>
        circuit.key === circuitKey,
    );

  if (circuitIndex !== -1) {
    cycleCircuitDefinitions.splice(
      circuitIndex,
      1,
    );
  }

  delete manualAssignments[
    circuitKey
  ];

  if (
    newRouteDraftCircuitKey.value ===
    circuitKey
  ) {
    newRouteDraftCircuitKey.value =
      "";
  }
}

async function selectRouteCreationMethod(
  method: RouteCreationMethod,
) {
  if (
    routeCreationMethod.value ===
    method
  ) {
    return;
  }

  const hasCurrentWork =
    currentRouteConnections.length >
      0 ||
    manualRouteNodes.length > 0 ||
    routeStart !== null ||
    routeEnd !== null;

  if (hasCurrentWork) {
    const shouldChange =
      window.confirm(
        "Já começaste a preparar um caminho.\n\n" +
          "Ao mudar o método de criação, o trabalho atual será descartado.\n\n" +
          "Queres continuar?",
      );

    if (!shouldChange) {
      return;
    }

    await discardCurrentRoute();

    routeStart = null;
    routeEnd = null;

    routeStartLabel.value =
      "nenhum";

    routeEndLabel.value =
      "nenhum";

    routeWaypoints.splice(0);

    manualRouteNodes.splice(0);

    isManualRouteRecording.value =
      false;
  }

  routeCreationMethod.value =
    method;

  flowMessage.value =
    method === "automatic"
      ? "Método por início e fim selecionado."
      : "Método tubo a tubo selecionado.";
}

async function createAutoRouteForSelectedCycleCircuit() {
  const circuitKey =
    createNewRouteCircuitDefinition();

  await createAutoRoute(
    circuitKey,
  );

  if (
    currentRouteConnections.length ===
    0
  ) {
    removeUnusedNewRouteCircuit(
      circuitKey,
    );

    return;
  }

  saveCycleCircuitDefinitionsToStorage();

  flowMessage.value =
    'Novo circuito "' +
    getNewRouteCircuitDisplayName() +
    '" preparado. Confirma e guarda o caminho.';
}

async function createManualRouteForSelectedCycleCircuit() {
  const circuitKey =
    createNewRouteCircuitDefinition();

  await createManualRouteFromSelection(
    circuitKey,
  );

  if (
    currentRouteConnections.length ===
    0
  ) {
    removeUnusedNewRouteCircuit(
      circuitKey,
    );

    return;
  }

  saveCycleCircuitDefinitionsToStorage();

  flowMessage.value =
    'Novo circuito "' +
    getNewRouteCircuitDisplayName() +
    '" preparado. Confirma e guarda o caminho.';
}

function getDefaultNewRouteCircuitName(
  kind: CycleCircuitKind,
) {
  if (kind === "hotSupply") {
    return "Ida quente";
  }

  if (kind === "coldSupply") {
    return "Ida fria";
  }

  if (kind === "hotReturn") {
    return "Retorno quente";
  }

  if (kind === "coldReturn") {
    return "Retorno frio";
  }

  return "Extra";
}

function getDefaultNewRouteCircuitColor(
  kind: CycleCircuitKind,
) {
  if (kind === "hotSupply") {
    return "#ff0000";
  }

  if (kind === "coldSupply") {
    return "#0077ff";
  }

  if (kind === "hotReturn") {
    return "#ff8c00";
  }

  if (kind === "coldReturn") {
    return "#7b1fa2";
  }

  return "#2e7d32";
}

function handleNewRouteCircuitKindChange() {
  newRouteCircuitName.value =
    getDefaultNewRouteCircuitName(
      newRouteCircuitKind.value,
    );

  newRouteCircuitColor.value =
    getDefaultNewRouteCircuitColor(
      newRouteCircuitKind.value,
    );

  newRouteDraftCircuitKey.value =
    "";
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
  setActiveIfcStorageItem(
    WATER_CYCLE_NAMES_STORAGE_KEY,
    JSON.stringify(cycleNames),
  );
}

function loadCycleNamesFromStorage() {
  const saved =
    getActiveIfcStorageItem(
      WATER_CYCLE_NAMES_STORAGE_KEY,
    );

  if (saved) {
    try {
      const parsed =
        JSON.parse(
          saved,
        ) as Record<
          string,
          string
        >;

      for (
        const [key, value] of
          Object.entries(parsed)
      ) {
        cycleNames[key] = value;
      }
    } catch (error) {
      console.error(
        "Erro ao carregar nomes dos ciclos:",
        error,
      );
    }
  }

    ensureCycleNames();
}

function toggleCycleNamesPanel() {
  isCycleNamesPanelOpen.value = !isCycleNamesPanelOpen.value;
}

function normalizeSavedRouteSearchText(
  value: string,
) {
  return value
    .toLowerCase()
    .normalize("NFD")
    .replace(
      /[\u0300-\u036f]/g,
      "",
    )
    .trim();
}

function getAllSavedRoutesOrdered() {
  return [...savedRoutes].sort(
    (
      firstRoute,
      secondRoute,
    ) =>
      firstRoute.name.localeCompare(
        secondRoute.name,
        "pt-PT",
        {
          sensitivity: "base",
          numeric: true,
        },
      ),
  );
}


function getFilteredSavedRoutes() {
  const searchText =
    normalizeSavedRouteSearchText(
      savedRouteSearchText.value,
    );

  const filteredRoutes =
    searchText
      ? savedRoutes.filter(
          (route) => {
            const routeSearchText =
              normalizeSavedRouteSearchText(
                [
                  route.name,
                  getSavedRouteGroupName(
                    route,
                  ),
                  getRouteCircuitDisplayLabel(
                    route,
                  ),
                ].join(" "),
              );

            return routeSearchText.includes(
              searchText,
            );
          },
        )
      : [...savedRoutes];

  return filteredRoutes.sort(
    (firstRoute, secondRoute) =>
      firstRoute.name.localeCompare(
        secondRoute.name,
        "pt",
        {
          sensitivity: "base",
          numeric: true,
        },
      ),
  );
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

function getSelectedManualValveKey() {
  const valveKey =
    selectedValveDesignationKey.value;

  if (!valveKey) {
    return "";
  }

  const valveNode =
    getValveNodeFromDesignationKey(
      valveKey,
    );

  if (!valveNode) {
    return "";
  }

  return valveKey;
}

function getManualValveControlledPipeCount() {
  const valveKey =
    getSelectedManualValveKey();

  if (!valveKey) {
    return 0;
  }

  return (
    manualValveControlledPipes.get(
      valveKey,
    )?.length ??
    0
  );
}

function getSelectedPipeNodesForManualValve() {
  const selectedPipes:
    ManualValveControlledPipe[] = [];

  for (
    const [modelId, localIds] of
      selectedItems
  ) {
    for (const localId of localIds) {
      const node = {
        modelId,
        localId,
      };

      if (
  isNodeInProtectedRoute(
    node,
  )
) {
  continue;
}

      if (
        !pipeTypeFlowNodes.has(
          nodeKey(node),
        )
      ) {
        continue;
      }

      selectedPipes.push({
        modelId,
        localId,
      });
    }
  }

  return selectedPipes;
}

async function clearManualValvePipeDraftHighlight() {
  const pipesByModel =
    new Map<
      string,
      number[]
    >();

  for (
    const pipe of
      highlightedManualValveDraftPipes.value
  ) {
    const localIds =
      pipesByModel.get(
        pipe.modelId,
      ) ?? [];

    localIds.push(
      pipe.localId,
    );

    pipesByModel.set(
      pipe.modelId,
      localIds,
    );
  }

  for (
    const [
      modelId,
      localIds,
    ] of pipesByModel
  ) {
    const model =
      loadedModels.get(
        modelId,
      );

    if (
      model &&
      localIds.length
    ) {
      await model.resetHighlight(
        localIds,
      );
    }
  }

  highlightedManualValveDraftPipes.value =
    [];

  await fragmentManager.core.update(
    true,
  );
}

async function highlightManualValvePipeDraft() {
  await clearManualValvePipeDraftHighlight();

  if (
    !manualValvePipeDraft.value.length
  ) {
    return;
  }

  const pipesByModel =
    new Map<
      string,
      number[]
    >();

  for (
    const pipe of
      manualValvePipeDraft.value
  ) {
    const localIds =
      pipesByModel.get(
        pipe.modelId,
      ) ?? [];

    localIds.push(
      pipe.localId,
    );

    pipesByModel.set(
      pipe.modelId,
      localIds,
    );
  }

  for (
    const [
      modelId,
      localIds,
    ] of pipesByModel
  ) {
    const model =
      loadedModels.get(
        modelId,
      );

    if (
      !model ||
      !localIds.length
    ) {
      continue;
    }

    await model.highlight(
      localIds,
      createHighlight(
        0xffb300,
        "manual-valve-controlled-pipes",
      ),
    );
  }

  highlightedManualValveDraftPipes.value =
    manualValvePipeDraft.value.map(
      (pipe) => ({
        modelId:
          pipe.modelId,

        localId:
          pipe.localId,
      }),
    );

  await fragmentManager.core.update(
    true,
  );
}

function startManualValvePipeDefinition() {
  const valveKey =
    getSelectedManualValveKey();

  if (!valveKey) {
    flowMessage.value =
      "Seleciona primeiro uma válvula.";

    window.alert(
      "Seleciona primeiro uma válvula no dropdown ou diretamente no modelo.",
    );

    return;
  }

  const savedPipes =
    manualValveControlledPipes.get(
      valveKey,
    ) ?? [];

  manualValvePipeDraft.value =
    savedPipes.map(
      (pipe) => ({
        modelId:
          pipe.modelId,

        localId:
          pipe.localId,
      }),
    );

  isManualValvePipeDefinitionMode.value =
    true;

  hasManualValvePipeDraftChanges.value =
    false;

  flowMessage.value =
    "Modo de definição dos tubos controlados ativo. Seleciona os tubos no modelo.";
}

async function cancelManualValvePipeDefinition() {
  const hasUnsavedChanges =
    hasManualValvePipeDraftChanges.value;

  if (hasUnsavedChanges) {
    const shouldCancel =
      window.confirm(
        "Existem alterações no rascunho que ainda não foram guardadas.\n\n" +
          "Queres cancelar e perder essas alterações?",
      );

    if (!shouldCancel) {
      return;
    }
  }

  manualValvePipeDraft.value =
    [];

  isManualValvePipeDefinitionMode.value =
    false;

  hasManualValvePipeDraftChanges.value =
    false;

  await clearManualValvePipeDraftHighlight();

  flowMessage.value =
    "Definição dos tubos controlados cancelada.";
}

function addSelectedPipesToManualValveDraft() {
  if (
    !isManualValvePipeDefinitionMode.value
  ) {
    return;
  }

  const selectedPipes =
    getSelectedPipeNodesForManualValve();

  if (!selectedPipes.length) {
    flowMessage.value =
      "Seleciona primeiro um ou mais tubos no modelo.";

    return;
  }

  const draftByNode =
    new Map<
      string,
      ManualValveControlledPipe
    >(
      manualValvePipeDraft.value.map(
        (pipe) => [
          nodeKey(pipe),
          pipe,
        ],
      ),
    );

  for (const pipe of selectedPipes) {
    draftByNode.set(
      nodeKey(pipe),
      {
        modelId:
          pipe.modelId,

        localId:
          pipe.localId,
      },
    );
  }

  manualValvePipeDraft.value = [
    ...draftByNode.values(),
  ];

  hasManualValvePipeDraftChanges.value =
    true;

  void highlightManualValvePipeDraft();

  flowMessage.value =
    selectedPipes.length +
    " tubo(s) adicionado(s) ao rascunho da válvula.";
}

function removeSelectedPipesFromManualValveDraft() {
  if (
    !isManualValvePipeDefinitionMode.value
  ) {
    return;
  }

  const selectedPipes =
    getSelectedPipeNodesForManualValve();

  if (!selectedPipes.length) {
    flowMessage.value =
      "Seleciona primeiro os tubos que queres retirar.";

    return;
  }

  const selectedPipeKeys =
    new Set(
      selectedPipes.map(
        (pipe) =>
          nodeKey(pipe),
      ),
    );

  const previousCount =
    manualValvePipeDraft.value.length;

  manualValvePipeDraft.value =
    manualValvePipeDraft.value.filter(
      (pipe) =>
        !selectedPipeKeys.has(
          nodeKey(pipe),
        ),
    );

  const removedCount =
    previousCount -
    manualValvePipeDraft.value.length;

  if (!removedCount) {
    flowMessage.value =
      "Os tubos selecionados não pertencem ao rascunho.";

    return;
  }

  hasManualValvePipeDraftChanges.value =
    true;

  void highlightManualValvePipeDraft();

  flowMessage.value =
    removedCount +
    " tubo(s) retirado(s) do rascunho.";
}

function clearManualValvePipeDraft() {
  if (
    !manualValvePipeDraft.value.length
  ) {
    flowMessage.value =
      "O rascunho já está vazio.";

    return;
  }

  const confirmed =
    window.confirm(
      "Queres remover todos os tubos do rascunho desta válvula?",
    );

  if (!confirmed) {
    return;
  }

  manualValvePipeDraft.value =
    [];

  void clearManualValvePipeDraftHighlight();

hasManualValvePipeDraftChanges.value =
  true;

  flowMessage.value =
    "Todos os tubos foram retirados do rascunho.";
}

async function saveManualValvePipeDefinition() {
  const valveKey =
    getSelectedManualValveKey();

  if (!valveKey) {
    window.alert(
      "Seleciona primeiro uma válvula.",
    );

    return;
  }

  if (
    !manualValvePipeDraft.value.length
  ) {
    window.alert(
      "Adiciona pelo menos um tubo antes de guardar a definição.",
    );

    return;
  }

  manualValveControlledPipes.set(
    valveKey,
    manualValvePipeDraft.value.map(
      (pipe) => ({
        modelId:
          pipe.modelId,

        localId:
          pipe.localId,
      }),
    ),
  );

  saveManualValveControlledPipesToStorage();

  hasManualValvePipeDraftChanges.value =
    false;

  isManualValvePipeDefinitionMode.value =
    false;

  await highlightManualValvePipeDraft();

  flowMessage.value =
    manualValvePipeDraft.value.length +
    " tubo(s) guardado(s) para a válvula selecionada.";
}

async function removeManualValvePipeDefinition() {
  const valveKey =
    getSelectedManualValveKey();

  if (!valveKey) {
    window.alert(
      "Seleciona primeiro uma válvula.",
    );

    return;
  }

  const savedPipes =
    manualValveControlledPipes.get(
      valveKey,
    );

  if (!savedPipes?.length) {
    flowMessage.value =
      "Esta válvula não tem uma definição manual guardada.";

    return;
  }

  const confirmed =
    window.confirm(
      "Queres remover todos os tubos controlados pela válvula selecionada?",
    );

  if (!confirmed) {
    return;
  }

  manualValveControlledPipes.delete(
    valveKey,
  );

  manualValvePipeDraft.value =
    [];

  saveManualValveControlledPipesToStorage();

  await clearManualValvePipeDraftHighlight();

  hasManualValvePipeDraftChanges.value =
    false;

  isManualValvePipeDefinitionMode.value =
    false;

  flowMessage.value =
    "A definição manual da válvula foi removida.";
}

async function highlightSavedManualValvePipes() {
  const valveKey =
    getSelectedManualValveKey();

  if (!valveKey) {
    window.alert(
      "Seleciona primeiro uma válvula.",
    );

    return;
  }

  const savedPipes =
    manualValveControlledPipes.get(
      valveKey,
    ) ?? [];

  if (!savedPipes.length) {
    flowMessage.value =
      "Esta válvula ainda não tem tubos controlados guardados.";

    return;
  }

  manualValvePipeDraft.value =
    savedPipes.map(
      (pipe) => ({
        modelId:
          pipe.modelId,

        localId:
          pipe.localId,
      }),
    );

  await highlightManualValvePipeDraft();

  flowMessage.value =
    savedPipes.length +
    " tubo(s) controlado(s) realçado(s).";
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
  setActiveIfcStorageItem(
    WATER_CYCLE_COUNT_STORAGE_KEY,
    String(
      waterCycleCount.value,
    ),
  );
}

function loadWaterCycleCountFromStorage() {
  const saved =
    getActiveIfcStorageItem(
      WATER_CYCLE_COUNT_STORAGE_KEY,
    );

  const parsed =
    saved
      ? Number(saved)
      : 3;

  const safeValue =
    Number.isFinite(parsed)
      ? Math.max(
          1,
          Math.min(
            12,
            Math.round(parsed),
          ),
        )
      : 3;

  waterCycleCount.value =
    safeValue;

  pendingWaterCycleCount.value =
    safeValue;

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

function getSavedRouteGroup(
  groupId?: string,
) {
  if (!groupId) {
    return null;
  }

  return (
    savedRouteGroups.find(
      (group) => group.id === groupId,
    ) ?? null
  );
}

function getSavedRouteGroupName(
  route: SavedRoute,
) {
  if (
    route.locked &&
    route.protectedSnapshot
  ) {
    return route
      .protectedSnapshot
      .circuitLabel;
  }

  if (
    route.locked &&
    route.protectedSnapshot
  ) {
    return route
      .protectedSnapshot
      .groupName;
  }

  return (
    getSavedRouteGroup(
      route.groupId,
    )?.name ??
    "Sem grupo"
  );
}

function getSavedRouteGroupColor(
  route: SavedRoute,
) {
  if (
  route.locked &&
  route.protectedSnapshot
) {
  return route
    .protectedSnapshot
    .effectiveColor;
}

  const group =
    getSavedRouteGroup(route.groupId);

  if (group) {
    return group.color;
  }

  if (route.customColor) {
    return route.customColor;
  }

  const circuit =
    getCycleCircuitDefinition(
      route.temperature,
    );

  if (circuit) {
    return getCycleCircuitDefaultColor(
      circuit,
    );
  }

  return getCircuitColorStyle(
    route.temperature,
  );
}

function shouldIncludeNodeInRouteSimulation(
  circuit: PipeCircuit,
  node: FlowNode,
) {
  if (
    selectedRouteIdsForSimulation.size === 0
  ) {
    return true;
  }

  return savedRoutes.some(
    (route) =>
      selectedRouteIdsForSimulation.has(
        route.id,
      ) &&
      route.temperature === circuit &&
      !route.hidden &&
      routeContainsAdaptedNode(
        route,
        node,
      ),
  );
}

async function toggleRouteForSimulation(
  routeId: string,
) {
    if (highlightedSavedRouteId.value) {
    flowMessage.value =
      "Limpa primeiro o realce do caminho antes de iniciar outra simulação.";

    return;
  }

  const route =
  savedRoutes.find(
    (savedRoute) =>
      savedRoute.id ===
      routeId,
  );

if (!route) {
  return;
}

if (
  route.needsDirectionRedefinition
) {
  window.alert(
    "Não é possível simular este caminho.\n\n" +
      "Foram adicionados tubos e o sentido tem de ser definido novamente.",
  );

  flowMessage.value =
    'Define novamente o sentido do caminho "' +
    route.name +
    '" antes de o simular.';

  return;
}

  isFlowing.value = false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value = false;

  const wasSelected =
    selectedRouteIdsForSimulation.has(
      routeId,
    );

  if (wasSelected) {
    selectedRouteIdsForSimulation.delete(
      routeId,
    );
  } else {
    selectedRouteIdsForSimulation.add(
      routeId,
    );
  }

  if (
    wasSelected &&
    selectedRouteIdsForSimulation.size === 0
  ) {
    isFlowing.value = false;

    isManualFlowAnimationRunning.value =
      false;

    isCentralSimulationRunning.value =
      false;

    isFlowManuallyPaused.value = true;

    await rebuildManualFlowLayer();

    flowMessage.value =
      "Nenhum caminho está selecionado para simulação. " +
      "Clica no botão da simulação geral para simular todos.";

    return;
  }

  flowMessage.value =
    "A preparar os caminhos selecionados.";

  await rebuildManualFlowLayer();

  if (
    hasFlowPreparationError.value ||
    !pipeParticles.length ||
    !isFlowAnimationReady.value
  ) {
    flowMessage.value =
      "Não foi possível iniciar a simulação dos caminhos selecionados.";

    return;
  }

  isFlowing.value = true;

  isManualFlowAnimationRunning.value =
    true;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value = false;

  flowMessage.value =
    selectedRouteIdsForSimulation.size +
    " caminho(s) em simulação.";
}

function isRouteSelectedForSimulation(
  routeId: string,
) {
  return (
    selectedRouteIdsForSimulation.has(
      routeId,
    )
  );
}

function toggleRouteSelectionForGrouping(
  routeId: string,
) {
  const selectedRoute =
    savedRoutes.find(
      (savedRoute) =>
        savedRoute.id ===
        routeId,
    );

  if (
    !selectedRoute ||
    selectedRoute.locked
  ) {
    selectedRouteIdsForGrouping.delete(
      routeId,
    );

    flowMessage.value =
      "Os caminhos protegidos não podem participar em operações em lote.";

    return;
  }

  if (
    selectedRouteIdsForGrouping.has(
      routeId,
    )
  ) {
    selectedRouteIdsForGrouping.delete(
      routeId,
    );

    return;
  }

  selectedRouteIdsForGrouping.add(
    routeId,
  );
}

function isRouteSelectedForGrouping(
  routeId: string,
) {
  return selectedRouteIdsForGrouping.has(
    routeId,
  );
}

function clearRouteGroupingSelection() {
  selectedRouteIdsForGrouping.clear();
}

async function deleteSelectedSavedRoutes() {
  const selectedRoutes =
    savedRoutes.filter(
      (route) =>
        selectedRouteIdsForGrouping.has(
          route.id,
        ),
    );

  if (!selectedRoutes.length) {
    flowMessage.value =
      "Seleciona primeiro pelo menos um percurso.";

    return;
  }

  const lockedRoutes =
    selectedRoutes.filter(
      (route) =>
        route.locked,
    );

  if (lockedRoutes.length) {
    const lockedRouteNames =
      lockedRoutes
        .map(
          (route) =>
            "• " + route.name,
        )
        .join("\n");

    window.alert(
      "Não é possível apagar a seleção porque contém percursos protegidos.\n\n" +
        lockedRouteNames +
        "\n\nDesprotege estes percursos antes de tentar novamente.",
    );

    flowMessage.value =
      "A eliminação foi bloqueada porque existem percursos protegidos.";

    return;
  }

  const routeNames =
    selectedRoutes
      .map(
        (route) =>
          "• " + route.name,
      )
      .join("\n");

  const mergedRouteCount =
    selectedRoutes.filter(
      (route) =>
        !!route.mergeBackup,
    ).length;

  const confirmationMessage =
    "Queres apagar os " +
    selectedRoutes.length +
    " percursos selecionados?\n\n" +
    routeNames +
    "\n\n" +
    (
      mergedRouteCount > 0
        ? mergedRouteCount +
          " percurso(s) resultam de uma união. " +
          "O histórico necessário para Desfazer união também será apagado.\n\n"
        : ""
    ) +
    "Esta ação não pode ser anulada.";

  const confirmed =
    window.confirm(
      confirmationMessage,
    );

  if (!confirmed) {
    flowMessage.value =
      "A eliminação dos percursos selecionados foi cancelada.";

    return;
  }

  const routeIdsToDelete =
    selectedRoutes.map(
      (route) =>
        route.id,
    );

  for (
    const routeId of
      routeIdsToDelete
  ) {
    await deleteSavedRoute(
      routeId,
      true,
      true,
    );
  }

  selectedRouteIdsForGrouping.clear();

  selectedSavedRouteIdForEditing.value =
    "";

  highlightedSavedRouteId.value =
    null;

  selectedItems.clear();

  selectedCount.value =
    0;

  if (modelHighlighter) {
    await modelHighlighter.clear(
      "saved-route-highlight",
    );

    await modelHighlighter.clear(
      "select",
    );
  }

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer(
      false,
    );
  } else {
    clearFlowVisuals();

    await fragmentManager.core.update(
      true,
    );
  }

  isFlowing.value =
    false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value =
    true;

  flowMessage.value =
    routeIdsToDelete.length +
    " percurso(s) selecionado(s) foram apagados.";
}

async function moveSelectedSavedRoutesToCycle() {
  const selectedRoutes = savedRoutes.filter(
    (route) =>
      selectedRouteIdsForGrouping.has(route.id),
  );

  if (!selectedRoutes.length) {
    flowMessage.value =
      "Seleciona primeiro pelo menos um caminho guardado.";
    return;
  }

  const lockedRoutes =
  selectedRoutes.filter(
    (route) =>
      route.locked,
  );

if (lockedRoutes.length > 0) {
  const lockedRouteNames =
    lockedRoutes
      .map(
        (route) =>
          route.name,
      )
      .join("\n");

  window.alert(
    "Não é possível mover caminhos protegidos para outro ciclo.\n\n" +
      "Desprotege primeiro:\n" +
      lockedRouteNames,
  );

  flowMessage.value =
    lockedRoutes.length === 1
      ? "O caminho selecionado está protegido."
      : lockedRoutes.length +
        " caminhos selecionados estão protegidos.";

  return;
}

  const targetCycleNumber = Number(
    targetCycleNumberForSavedRoutes.value,
  );

  if (
    !Number.isFinite(targetCycleNumber) ||
    targetCycleNumber < 1 ||
    targetCycleNumber > waterCycleCount.value
  ) {
    flowMessage.value =
      "Seleciona um ciclo de destino válido.";
    return;
  }

  let movedCount = 0;

  for (const route of selectedRoutes) {
    const sourceCircuit =
      getCycleCircuitDefinition(
        route.temperature,
      );

    if (!sourceCircuit) {
      continue;
    }

    if (
      sourceCircuit.cycleNumber ===
      targetCycleNumber
    ) {
      continue;
    }

    const routesUsingSourceCircuit =
      savedRoutes.filter(
        (savedRoute) =>
          savedRoute.temperature ===
          sourceCircuit.key,
      );

    const allSourceRoutesAreSelected =
      routesUsingSourceCircuit.every(
        (savedRoute) =>
          selectedRouteIdsForGrouping.has(
            savedRoute.id,
          ),
      );

    if (allSourceRoutesAreSelected) {
      sourceCircuit.cycleNumber =
        targetCycleNumber;

      movedCount++;
      continue;
    }

    const newCircuitKey =
      "cycle" +
      targetCycleNumber +
      "-" +
      sourceCircuit.kind +
      "-" +
      crypto.randomUUID();

    const newCircuit: CycleCircuitDefinition = {
      key: newCircuitKey,
      cycleNumber: targetCycleNumber,
      kind: sourceCircuit.kind,
      name: sourceCircuit.name,
      color: sourceCircuit.color,
      defaultColor:
        sourceCircuit.defaultColor ??
        sourceCircuit.color,
      locked: false,
    };

    cycleCircuitDefinitions.push(
      newCircuit,
    );

    manualAssignments[newCircuitKey] =
      new Map();

    for (const node of route.path) {
      const oldAssignmentSet =
        getAssignmentSet(
          sourceCircuit.key,
          node.modelId,
        );

      oldAssignmentSet.delete(
        node.localId,
      );

      getAssignmentSet(
        newCircuitKey,
        node.modelId,
      ).add(node.localId);
    }

    route.temperature = newCircuitKey;

    movedCount++;
  }

  saveCycleCircuitDefinitionsToStorage();
  saveRoutesToStorage();

  circuitMaterialCache.clear();

  activeCycleNumber.value =
    targetCycleNumber;

  selectDefaultCircuitForActiveCycle();

  updateManualStats();

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer();
  }

  clearRouteGroupingSelection();

  flowMessage.value =
    movedCount > 0
      ? movedCount +
        " caminho(s) movido(s) para " +
        getCycleDisplayName(
          targetCycleNumber,
        ) +
        "."
      : "Os caminhos selecionados já pertencem ao ciclo escolhido.";
}

function openIndividualRouteColorDialog(
  route: SavedRoute,
) {
  if (route.groupId) {
    flowMessage.value =
      "Este caminho pertence a um grupo. Altera a cor do grupo.";

    return;
  }

  if (route.locked) {
    flowMessage.value =
      "O caminho \"" +
      route.name +
      "\" está protegido. Desprotege primeiro para alterar a cor.";

    return;
  }

  editingRouteColorId.value =
    route.id;

  pendingIndividualRouteColor.value =
  getSavedRouteGroupColor(route);

  isRouteColorDialogOpen.value =
    true;
}

function closeIndividualRouteColorDialog() {
  isRouteColorDialogOpen.value =
    false;

  editingRouteColorId.value =
    null;
}

async function saveIndividualRouteColor() {
  const route = savedRoutes.find(
    (savedRoute) =>
      savedRoute.id ===
      editingRouteColorId.value,
  );

  if (!route) {
    flowMessage.value =
      "Não foi possível encontrar o caminho.";

    return;
  }

  if (route.groupId) {
    flowMessage.value =
      "Este caminho pertence a um grupo.";

    return;
  }

  if (route.locked) {
    flowMessage.value =
      "Desprotege primeiro o caminho.";

    return;
  }

  const selectedColor =
    pendingIndividualRouteColor.value.trim();

  if (
    !/^#[0-9a-fA-F]{6}$/.test(
      selectedColor,
    )
  ) {
    flowMessage.value =
      "Seleciona uma cor válida.";

    return;
  }

  route.customColor = selectedColor;

  saveRoutesToStorage();

  circuitMaterialCache.clear();

  isRouteColorDialogOpen.value =
    false;

  editingRouteColorId.value =
    null;

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value =
    "A cor do caminho \"" +
    route.name +
    "\" foi alterada.";
}

async function resetIndividualRouteColor(
  route: SavedRoute,
) {
  if (route.groupId) {
    flowMessage.value =
      "Retira primeiro o caminho do grupo para repor a cor individual.";

    return;
  }

  if (route.locked) {
    flowMessage.value =
      "O caminho está protegido. Desprotege primeiro para repor a cor.";

    return;
  }

  delete route.customColor;

  saveRoutesToStorage();

  circuitMaterialCache.clear();

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value =
    "A cor original do caminho \"" +
    route.name +
    "\" foi reposta.";
}

function openRouteGroupColorDialog(
  route: SavedRoute,
) {
  const group =
    getSavedRouteGroup(route.groupId);

  if (!group) {
    flowMessage.value =
      "Este caminho não pertence a nenhum grupo.";

    return;
  }

  editingRouteGroupId.value =
    group.id;

  pendingRouteGroupName.value =
    group.name;

  pendingRouteGroupColor.value =
    group.color;

  isRouteGroupDialogOpen.value =
    true;
}

function openRouteGroupingDialog() {
  editingRouteGroupId.value = null;

  const selectedRoutes = savedRoutes.filter(
    (route) =>
      selectedRouteIdsForGrouping.has(
        route.id,
      ),
  );

  if (selectedRoutes.length < 2) {
    flowMessage.value =
      "Seleciona pelo menos dois caminhos para agrupar.";
    return;
  }

  const firstRoute = selectedRoutes[0];

  pendingRouteGroupName.value =
    "Novo grupo de caminhos";

  pendingRouteGroupColor.value =
    getSavedRouteGroupColor(firstRoute);

  isRouteGroupDialogOpen.value = true;
}

function closeRouteGroupingDialog() {
  isRouteGroupDialogOpen.value =
    false;

  editingRouteGroupId.value =
    null;
}

async function saveRouteGroupChanges() {
  if (!editingRouteGroupId.value) {
    await groupSelectedSavedRoutes();
    return;
  }

  const group =
    savedRouteGroups.find(
      (savedGroup) =>
        savedGroup.id ===
        editingRouteGroupId.value,
    );

  if (!group) {
    flowMessage.value =
      "Não foi possível encontrar o grupo.";

    return;
  }

  const groupName =
    pendingRouteGroupName.value.trim();

  const groupColor =
    pendingRouteGroupColor.value.trim();

  if (!groupName) {
    flowMessage.value =
      "Indica um nome para o grupo.";

    return;
  }

  if (
    !/^#[0-9a-fA-F]{6}$/.test(
      groupColor,
    )
  ) {
    flowMessage.value =
      "Seleciona uma cor válida.";

    return;
  }

  group.name = groupName;
  group.color = groupColor;

  circuitMaterialCache.clear();

  saveRouteGroupsToStorage();
saveRoutesToStorage();

  isRouteGroupDialogOpen.value = false;
  editingRouteGroupId.value = null;

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value =
    "A cor do grupo \"" +
    groupName +
    "\" foi alterada.";
}

async function groupSelectedSavedRoutes() {
  const selectedRoutes =
    savedRoutes.filter(
      (route) =>
        selectedRouteIdsForGrouping.has(
          route.id,
        ),
    );

  if (selectedRoutes.length < 2) {
    flowMessage.value =
      "Seleciona pelo menos dois percursos para unir.";

    return;
  }

  const lockedRoutes =
    selectedRoutes.filter(
      (route) => route.locked,
    );

  if (lockedRoutes.length > 0) {
  const lockedRoutesMessage =
    "Não é possível unir percursos protegidos.\n\n" +
    "Desprotege primeiro:\n" +
    lockedRoutes
      .map((route) => route.name)
      .join("\n");

  window.alert(
    lockedRoutesMessage,
  );

  flowMessage.value =
    lockedRoutesMessage;

  return;
}

  const sourceCircuit =
    selectedRoutes[0].temperature;

  const sourceCircuitKeys =
    new Set(
      selectedRoutes.map(
        (route) =>
          route.temperature,
      ),
    );

  const mergedRouteName =
    pendingRouteGroupName.value.trim();

  const mergedRouteColor =
    pendingRouteGroupColor.value.trim();

  if (!mergedRouteName) {
    flowMessage.value =
      "Indica um nome para o novo percurso.";

    return;
  }

  if (
    !/^[#][0-9a-fA-F]{6}$/.test(
      mergedRouteColor,
    )
  ) {
    flowMessage.value =
      "Seleciona uma cor válida.";

    return;
  }

  const sourceRouteIds =
    new Set(
      selectedRoutes.map(
        (route) => route.id,
      ),
    );

  const mergedPathByKey =
    new Map<string, FlowNode>();

  for (const route of selectedRoutes) {
    for (const node of route.path) {
      mergedPathByKey.set(
        nodeKey(node),
        {
          modelId: node.modelId,
          localId: node.localId,
        },
      );
    }
  }

  const mergedPath = [
    ...mergedPathByKey.values(),
  ];

  if (mergedPath.length < 2) {
    flowMessage.value =
      "Não existem tubos suficientes para criar o percurso unido.";

    return;
  }

  const mergedStartsByKey =
    new Map<string, FlowNode>();

  const mergedEndsByKey =
    new Map<string, FlowNode>();

  for (const route of selectedRoutes) {
    for (
      const startNode of
        route.directionStarts ?? []
    ) {
      mergedStartsByKey.set(
        nodeKey(startNode),
        {
          modelId:
            startNode.modelId,
          localId:
            startNode.localId,
        },
      );
    }

    for (
      const endNode of
        route.directionEnds ?? []
    ) {
      mergedEndsByKey.set(
        nodeKey(endNode),
        {
          modelId:
            endNode.modelId,
          localId:
            endNode.localId,
        },
      );
    }
  }

  const sourceValveLinks =
    [
      ...valveControlledPipeLinks.entries(),
    ]
      .filter(([, linkedPipes]) =>
        linkedPipes.some(
          (linkedPipe) =>
            sourceRouteIds.has(
              linkedPipe.routeId,
            ),
        ),
      )
      .map(
        ([
          valveKey,
          linkedPipes,
        ]) => ({
          valveKey,

          linkedPipes:
            linkedPipes.map(
              (linkedPipe) => ({
                ...linkedPipe,
              }),
            ),
        }),
      );

  const sourceBlockedValveLinks =
  [
    ...valveBlockedPipeLinks.entries(),
  ]
    .filter(([, linkedPipes]) =>
      linkedPipes.some(
        (linkedPipe) =>
          sourceRouteIds.has(
            linkedPipe.routeId,
          ),
      ),
    )
    .map(
      ([
        valveKey,
        linkedPipes,
      ]) => ({
        valveKey,

        linkedPipes:
          linkedPipes.map(
            (linkedPipe) => ({
              ...linkedPipe,
            }),
          ),
      }),
    );

  const sourceSimulationRouteIds =
    selectedRoutes
      .filter((route) =>
        selectedRouteIdsForSimulation.has(
          route.id,
        ),
      )
      .map((route) => route.id);

  const sourceRoutes:
    SavedRouteMergeSource[] =
    selectedRoutes.map(
      (route) => ({
        id: route.id,
        name: route.name,
        temperature:
          route.temperature,

        path: route.path.map(
          (node) => ({
            modelId: node.modelId,
            localId: node.localId,
          }),
        ),

        hidden: route.hidden,
        locked: route.locked,
        groupId: route.groupId,

        originalCircuitColor:
          route.originalCircuitColor,

        customColor:
          route.customColor,

        directionStarts:
          route.directionStarts?.map(
            (node) => ({
              modelId: node.modelId,
              localId: node.localId,
            }),
          ),

        directionEnds:
          route.directionEnds?.map(
            (node) => ({
              modelId: node.modelId,
              localId: node.localId,
            }),
          ),

      mergeBackup:
  route.mergeBackup
    ? JSON.parse(
        JSON.stringify(
          route.mergeBackup,
        ),
      )
    : undefined,
      }),
    );

  const mergedRouteId =
    crypto.randomUUID();

  const mergedRoute: SavedRoute = {
    id: mergedRouteId,
    name: mergedRouteName,
    temperature: sourceCircuit,
    path: mergedPath,
    hidden: false,
    locked: false,
    customColor:
      mergedRouteColor,

    directionStarts:
      [
        ...mergedStartsByKey.values(),
      ],

    directionEnds:
      [
        ...mergedEndsByKey.values(),
      ],

    mergeBackup: {
  sourceRoutes,
  sourceValveLinks,
  sourceBlockedValveLinks,
  sourceSimulationRouteIds,
},
  };

  for (
    let routeIndex =
      savedRoutes.length - 1;
    routeIndex >= 0;
    routeIndex--
  ) {
    if (
      sourceRouteIds.has(
        savedRoutes[routeIndex].id,
      )
    ) {
      savedRoutes.splice(
        routeIndex,
        1,
      );
    }
  }

  savedRoutes.push(
    mergedRoute,
  );

  for (const node of mergedPath) {
  for (
    const circuitKey of
      sourceCircuitKeys
  ) {
    getAssignmentSet(
      circuitKey,
      node.modelId,
    ).delete(
      node.localId,
    );
  }

  getAssignmentSet(
    sourceCircuit,
    node.modelId,
  ).add(
    node.localId,
  );
}

  for (
    const [
      valveKey,
      linkedPipes,
    ] of valveControlledPipeLinks
  ) {
    const updatedLinkedPipes =
      linkedPipes.map(
        (linkedPipe) =>
          sourceRouteIds.has(
            linkedPipe.routeId,
          )
            ? {
                ...linkedPipe,
                routeId:
                  mergedRouteId,
              }
            : linkedPipe,
      );

    valveControlledPipeLinks.set(
      valveKey,
      updatedLinkedPipes,
    );
  }

  for (
    const [
      valveKey,
      linkedPipes,
    ] of valveBlockedPipeLinks
  ) {
    const updatedLinkedPipes =
      linkedPipes.map(
        (linkedPipe) =>
          sourceRouteIds.has(
            linkedPipe.routeId,
          )
            ? {
                ...linkedPipe,
                routeId:
                  mergedRouteId,
              }
            : linkedPipe,
      );

    valveBlockedPipeLinks.set(
      valveKey,
      updatedLinkedPipes,
    );
  }

  for (
    const sourceRouteId of
      sourceRouteIds
  ) {
    selectedRouteIdsForSimulation.delete(
      sourceRouteId,
    );

    selectedRouteIdsForGrouping.delete(
      sourceRouteId,
    );
  }

  if (
    sourceSimulationRouteIds.length > 0
  ) {
    selectedRouteIdsForSimulation.add(
      mergedRouteId,
    );
  }

  circuitMaterialCache.clear();

  saveRoutesToStorage();
  saveValvePipeLinksToStorage();

  clearRouteGroupingSelection();

  isRouteGroupDialogOpen.value =
    false;

  editingRouteGroupId.value =
    null;

  if (
    countAssignments() > 0 ||
    flowConnections.length > 0
  ) {
    await rebuildManualFlowLayer();
  }

  flowMessage.value =
    selectedRoutes.length +
    ' percursos unidos em "' +
    mergedRouteName +
    '". O novo percurso contém ' +
    mergedPath.length +
    " tubo(s) sem duplicados.";
}

function getSelectedSavedRouteForDirection() {
  if (!selectedSavedRouteDirectionId.value) {
    return null;
  }

  return (
    savedRoutes.find(
      (route) =>
        route.id ===
        selectedSavedRouteDirectionId.value,
    ) ?? null
  );
}

async function openSavedRouteDirectionPanel(
  route: SavedRoute,
) {
  if (route.locked) {
    flowMessage.value =
      "O caminho \"" +
      route.name +
      "\" está protegido. Desprotege primeiro para alterar o sentido.";

    return;
  }

  isRouteDirectionFocusModeActive.value =
  true;

flowPreparationRunId++;

isPreparingFlowAnimation.value =
  false;

isFlowAnimationReady.value =
  false;

flowPreparationProgress.value =
  0;

hasFlowPreparationError.value =
  false;

  isApplyingSavedRouteHighlight.value =
    true;

  try {
    stopFlowForSavedRouteHighlight();

    selectedSavedRouteDirectionId.value =
      route.id;

      savedRouteDirectionWarning.value =
  "";

    savedRouteDirectionStarts.value =
      route.directionStarts
        ? route.directionStarts.map(
            (node) => ({
              modelId: node.modelId,
              localId: node.localId,
            }),
          )
        : [];

    savedRouteDirectionEnds.value =
      route.directionEnds
        ? route.directionEnds.map(
            (node) => ({
              modelId: node.modelId,
              localId: node.localId,
            }),
          )
        : [];

    isSavedRouteDirectionPanelOpen.value =
      true;

    highlightedSavedRouteId.value =
      route.id;

    await enforceActiveSavedRouteHighlight();

    flowMessage.value =
      "Seleciona os tubos de início e de fim do caminho.";
  } finally {
    isApplyingSavedRouteHighlight.value =
      false;
  }
}

async function closeSavedRouteDirectionPanel(
  shouldRestoreFlow = true,
) {
  isApplyingSavedRouteHighlight.value =
    true;

  try {
    isSavedRouteDirectionPanelOpen.value =
      false;

    selectedSavedRouteDirectionId.value =
      "";

    savedRouteDirectionStarts.value =
      [];

    savedRouteDirectionEnds.value =
      [];

    savedRouteDirectionWarning.value =
  "";

    isRouteDirectionFocusModeActive.value =
      false;

    highlightedSavedRouteId.value =
      null;

    selectedItems.clear();

    selectedCount.value =
      0;

    selectedTubeRouteInfo.value =
      "";

    selectedIfcDetailsText.value =
      "";

    isIfcDetailsPanelOpen.value =
      false;

    if (modelHighlighter) {
      await modelHighlighter.clear(
        "saved-route-highlight",
      );

      await modelHighlighter.clear(
        "select",
      );
    }

    await clearPersistentCircuitHighlights();

    for (
      const model of
        loadedModels.values()
    ) {
      await model.resetHighlight();
    }

    if (shouldRestoreFlow) {
      if (
        countAssignments() > 0 ||
        flowConnections.length > 0
      ) {
        await rebuildManualFlowLayer(
          false,
        );
      } else {
        await fragmentManager.core.update(
          true,
        );
      }

      isFlowing.value =
        false;

      isManualFlowAnimationRunning.value =
        false;

      isCentralSimulationRunning.value =
        false;

      isFlowManuallyPaused.value =
        true;

      flowMessage.value =
        "Definição do sentido cancelada.";
    }
  } finally {
    isApplyingSavedRouteHighlight.value =
      false;
  }
}

function getSelectedDirectionNodeFromRoute() {
  const route =
    getSelectedSavedRouteForDirection();

  if (!route) {
    flowMessage.value =
      "Seleciona primeiro um caminho guardado.";

    return null;
  }

  const selectedNode =
    getFirstSelectedNode();

  if (!selectedNode) {
    flowMessage.value =
      "Seleciona primeiro um tubo no modelo.";

    return null;
  }

  const belongsToRoute = route.path.some(
    (node) =>
      isSameNode(node, selectedNode),
  );

  if (!belongsToRoute) {
    flowMessage.value =
      "O elemento selecionado não pertence ao caminho \"" +
      route.name +
      "\".";

    return null;
  }

  return {
    modelId: selectedNode.modelId,
    localId: selectedNode.localId,
  };
}

function hasDirectionNode(
  nodes: FlowNode[],
  node: FlowNode,
) {
  return nodes.some(
    (existingNode) =>
      existingNode.modelId === node.modelId &&
      existingNode.localId === node.localId,
  );
}

function setSavedRouteDirectionStart() {
  const selectedNode =
    getSelectedDirectionNodeFromRoute();

  if (!selectedNode) {
    return;
  }

  if (
    hasDirectionNode(
      savedRouteDirectionStarts.value,
      selectedNode,
    )
  ) {
    flowMessage.value =
      "Este tubo já está definido como início.";

    return;
  }

  const isAlreadyAnEnd =
    hasDirectionNode(
      savedRouteDirectionEnds.value,
      selectedNode,
    );

  if (isAlreadyAnEnd) {
    flowMessage.value =
      "O mesmo tubo não pode ser simultaneamente início e fim.";

    return;
  }

  savedRouteDirectionStarts.value.push({
    modelId: selectedNode.modelId,
    localId: selectedNode.localId,
  });

  flowMessage.value =
    "Início adicionado: tubo #" +
    selectedNode.localId +
    ".";
}

function setSavedRouteDirectionEnd() {
  const selectedNode =
    getSelectedDirectionNodeFromRoute();

  if (!selectedNode) {
    return;
  }

  if (
    hasDirectionNode(
      savedRouteDirectionEnds.value,
      selectedNode,
    )
  ) {
    flowMessage.value =
      "Este tubo já está definido como fim.";

    return;
  }

  const isAlreadyAStart =
    hasDirectionNode(
      savedRouteDirectionStarts.value,
      selectedNode,
    );

  if (isAlreadyAStart) {
    flowMessage.value =
      "O mesmo tubo não pode ser simultaneamente início e fim.";

    return;
  }

  savedRouteDirectionEnds.value.push({
    modelId: selectedNode.modelId,
    localId: selectedNode.localId,
  });

  flowMessage.value =
    "Fim adicionado: tubo #" +
    selectedNode.localId +
    ".";
}

function removeSavedRouteDirectionStart(
  nodeToRemove: FlowNode,
) {
  savedRouteDirectionStarts.value =
    savedRouteDirectionStarts.value.filter(
      (node) =>
        !(
          node.modelId ===
            nodeToRemove.modelId &&
          node.localId ===
            nodeToRemove.localId
        ),
    );
}

function removeSavedRouteDirectionEnd(
  nodeToRemove: FlowNode,
) {
  savedRouteDirectionEnds.value =
    savedRouteDirectionEnds.value.filter(
      (node) =>
        !(
          node.modelId ===
            nodeToRemove.modelId &&
          node.localId ===
            nodeToRemove.localId
        ),
    );
}

async function findPathInsideSavedRoute(
  route: SavedRoute,
  start: FlowNode,
  end: FlowNode,
) {
  if (
    start.modelId !==
    end.modelId
  ) {
    return [] as FlowNode[];
  }

  const model =
    loadedModels.get(
      start.modelId,
    );

  if (!model) {
    return [] as FlowNode[];
  }

  const routeNodes =
    route.path.filter(
      (node) =>
        node.modelId ===
        start.modelId,
    );

  const localIds = [
    ...new Set(
      routeNodes.map(
        (node) =>
          node.localId,
      ),
    ),
  ];

  const boxes =
    await model.getBoxes(
      localIds,
    );

  const graphItems =
    new Map<
      string,
      PipeGraphItem
    >();

  for (
    let index = 0;
    index < localIds.length;
    index++
  ) {
    const graphItem =
      graphItemFromBox(
        start.modelId,
        localIds[index],
        boxes[index],
      );

    if (!graphItem) {
      continue;
    }

    graphItems.set(
      nodeKey(
        graphItem,
      ),
      graphItem,
    );
  }

  const graph =
    buildPipeGraph(
      graphItems,
    );

  const pathKeys =
    shortestPath(
      graph,
      nodeKey(
        start,
      ),
      nodeKey(
        end,
      ),
    );

  const result:
    FlowNode[] = [];

  for (const key of pathKeys) {
    const item =
      graphItems.get(
        key,
      );

    if (!item) {
      continue;
    }

    result.push({
      modelId:
        item.modelId,

      localId:
        item.localId,
    });
  }

  return result;
}

async function findSavedRouteDirectionPaths(
  route: SavedRoute,
  starts: FlowNode[],
  ends: FlowNode[],
) {
  const directionPaths: FlowNode[][] = [];

  for (const start of starts) {
    for (const end of ends) {
      const path =
        await findPathInsideSavedRoute(
          route,
          start,
          end,
        );

      if (!path.length) {
        continue;
      }

      directionPaths.push(path);
    }
  }

  return directionPaths;
}

function selectCompatibleDirectionPaths(
  directionPaths: FlowNode[][],
) {
  const compatiblePaths:
    FlowNode[][] = [];

  const rejectedPaths:
    FlowNode[][] = [];

  const directedEdges =
    new Set<string>();

  const knownPathSignatures =
    new Set<string>();

  const orderedPaths = [
    ...directionPaths,
  ].sort(
    (
      firstPath,
      secondPath,
    ) =>
      firstPath.length -
      secondPath.length,
  );

  for (const path of orderedPaths) {
    const pathSignature =
      path
        .map(
          (node) =>
            nodeKey(node),
        )
        .join(">");

    if (
      knownPathSignatures.has(
        pathSignature,
      )
    ) {
      continue;
    }

    knownPathSignatures.add(
      pathSignature,
    );

    let hasConflict =
      false;

    for (
      let nodeIndex = 0;
      nodeIndex <
        path.length - 1;
      nodeIndex++
    ) {
      const currentNode =
        path[nodeIndex];

      const nextNode =
        path[nodeIndex + 1];

      const reverseEdgeKey =
        nodeKey(nextNode) +
        "->" +
        nodeKey(currentNode);

      if (
        directedEdges.has(
          reverseEdgeKey,
        )
      ) {
        hasConflict =
          true;

        break;
      }
    }

    if (hasConflict) {
      rejectedPaths.push(
        path,
      );

      continue;
    }

    compatiblePaths.push(
      path,
    );

    for (
      let nodeIndex = 0;
      nodeIndex <
        path.length - 1;
      nodeIndex++
    ) {
      const currentNode =
        path[nodeIndex];

      const nextNode =
        path[nodeIndex + 1];

      directedEdges.add(
        nodeKey(currentNode) +
          "->" +
          nodeKey(nextNode),
      );
    }
  }

  const ambiguousNodes =
    new Map<
      string,
      FlowNode
    >();

  for (
    const rejectedPath of
      rejectedPaths
  ) {
    for (
      const node of
        rejectedPath
    ) {
      ambiguousNodes.set(
        nodeKey(node),
        {
          modelId:
            node.modelId,

          localId:
            node.localId,
        },
      );
    }
  }

  return {
    compatiblePaths,
    rejectedPaths,

    ambiguousNodes: [
      ...ambiguousNodes.values(),
    ],
  };
}

function applySavedRouteDirectionPaths(
  route: SavedRoute,
  directionPaths: FlowNode[][],
) {
  const routeNodeKeys = new Set(
    route.path.map((node) =>
      automaticDirectionKey(
        route.temperature,
        node,
      ),
    ),
  );

  for (const key of routeNodeKeys) {
    automaticDirectionNeighbors.delete(
      key,
    );
  }

  const directedEdges =
    new Set<string>();

  const rejectedReverseEdges =
    new Set<string>();

  for (const path of directionPaths) {
    for (
      let nodeIndex = 0;
      nodeIndex < path.length - 1;
      nodeIndex++
    ) {
      const currentNode =
        path[nodeIndex];

      const nextNode =
        path[nodeIndex + 1];

      const currentNodeKey =
        nodeKey(currentNode);

      const nextNodeKey =
        nodeKey(nextNode);

      const edgeKey =
        currentNodeKey +
        "->" +
        nextNodeKey;

      const reverseEdgeKey =
        nextNodeKey +
        "->" +
        currentNodeKey;

      if (
        directedEdges.has(
          reverseEdgeKey,
        )
      ) {
        rejectedReverseEdges.add(
          edgeKey,
        );

        continue;
      }

      directedEdges.add(
        edgeKey,
      );

      const currentDirectionKey =
        automaticDirectionKey(
          route.temperature,
          currentNode,
        );

      const nextDirectionKey =
        automaticDirectionKey(
          route.temperature,
          nextNode,
        );

      const currentDirection =
        automaticDirectionNeighbors.get(
          currentDirectionKey,
        ) ?? {
          previous: null,
          next: null,
        };

      const nextDirection =
        automaticDirectionNeighbors.get(
          nextDirectionKey,
        ) ?? {
          previous: null,
          next: null,
        };

      if (!currentDirection.next) {
        currentDirection.next = {
          modelId:
            nextNode.modelId,
          localId:
            nextNode.localId,
        };
      }

      if (!nextDirection.previous) {
        nextDirection.previous = {
          modelId:
            currentNode.modelId,
          localId:
            currentNode.localId,
        };
      }

      automaticDirectionNeighbors.set(
        currentDirectionKey,
        currentDirection,
      );

      automaticDirectionNeighbors.set(
        nextDirectionKey,
        nextDirection,
      );
    }
  }

  if (rejectedReverseEdges.size > 0) {
    console.warn(
      "Foram ignoradas ligações com sentidos contraditórios no percurso:",
      route.name,
      [...rejectedReverseEdges],
    );
  }
}

async function applyAndSaveSavedRouteDirection() {
  const route =
    getSelectedSavedRouteForDirection();

  if (!route) {
    flowMessage.value =
      "Não foi possível encontrar o caminho selecionado.";

    return;
  }

  if (route.locked) {
    flowMessage.value =
      "Desprotege o caminho antes de alterar o sentido.";

    return;
  }

  if (
    savedRouteDirectionStarts.value.length ===
    0
  ) {
    flowMessage.value =
      "Define pelo menos um início.";

    return;
  }

  if (
    savedRouteDirectionEnds.value.length ===
    0
  ) {
    flowMessage.value =
      "Define pelo menos um fim.";

    return;
  }

  savedRouteDirectionWarning.value =
  "";

const directionPaths =
  await findSavedRouteDirectionPaths(
    route,
    savedRouteDirectionStarts.value,
    savedRouteDirectionEnds.value,
  );

const expectedCombinationCount =
  savedRouteDirectionStarts.value
    .length *
  savedRouteDirectionEnds.value
    .length;

const unconnectedCombinationCount =
  Math.max(
    0,
    expectedCombinationCount -
      directionPaths.length,
  );

if (!directionPaths.length) {
  const message =
    "Não foi encontrada uma ligação entre os inícios e os fins definidos.";

  savedRouteDirectionWarning.value =
    message;

  flowMessage.value =
    message;

  window.alert(
    message,
  );

  return;
}

const {
  compatiblePaths,
  rejectedPaths,
  ambiguousNodes,
} =
  selectCompatibleDirectionPaths(
    directionPaths,
  );

if (!compatiblePaths.length) {
  const message =
    "Não foi possível encontrar nenhuma ligação com um sentido compatível.";

  savedRouteDirectionWarning.value =
    message;

  flowMessage.value =
    message;

  window.alert(
    message,
  );

  return;
}

const warningParts:
  string[] = [];

if (rejectedPaths.length > 0) {
  warningParts.push(
    rejectedPaths.length +
      " ligação(ões) ambígua(s) foram ignoradas por usarem tubos em sentidos opostos.",
  );
}

if (
  unconnectedCombinationCount > 0
) {
  warningParts.push(
    unconnectedCombinationCount +
      " combinação(ões) de início e fim não tinham ligação.",
  );
}

if (ambiguousNodes.length > 0) {
  const ambiguousPipeLabels =
    ambiguousNodes
      .slice(0, 8)
      .map(
        (node) =>
          "#" + node.localId,
      )
      .join(", ");

  warningParts.push(
    "Tubos envolvidos em ligações ambíguas: " +
      ambiguousPipeLabels +
      (
        ambiguousNodes.length > 8
          ? " e mais " +
            (
              ambiguousNodes.length -
              8
            )
          : ""
      ) +
      ".",
  );
}

savedRouteDirectionWarning.value =
  warningParts.join(" ");

  route.directionStarts =
    savedRouteDirectionStarts.value.map(
      (node) => ({
        modelId: node.modelId,
        localId: node.localId,
      }),
    );

  route.directionEnds =
    savedRouteDirectionEnds.value.map(
      (node) => ({
        modelId: node.modelId,
        localId: node.localId,
      }),
    );

  route.needsDirectionRedefinition =
  false;

  delete route.protectedDirections;

  applySavedRouteDirectionPaths(
  route,
  compatiblePaths,
);

  for (const node of route.path) {
    const reversedIds =
      reversedPipeDirections.get(
        node.modelId,
      );

    reversedIds?.delete(
      node.localId,
    );

    if (
      reversedIds &&
      reversedIds.size === 0
    ) {
      reversedPipeDirections.delete(
        node.modelId,
      );
    }

    const syncedIds =
      syncedPipeDirections.get(
        node.modelId,
      );

    syncedIds?.delete(
      node.localId,
    );

    if (
      syncedIds &&
      syncedIds.size === 0
    ) {
      syncedPipeDirections.delete(
        node.modelId,
      );
    }
  }

  saveReversedDirectionsToStorage();
  saveSyncedPipeDirectionsToStorage();

  saveRoutesToStorage();
  saveReversedDirectionsToStorage();
  saveSyncedPipeDirectionsToStorage();

  await closeSavedRouteDirectionPanel(
  false,
);

await rebuildManualFlowLayer(
  false,
);

highlightedSavedRouteId.value =
  null;

selectedItems.clear();

selectedCount.value =
  0;

if (modelHighlighter) {
  await modelHighlighter.clear(
    "saved-route-highlight",
  );

  await modelHighlighter.clear(
    "select",
  );
}

await fragmentManager.core.update(
  true,
);

isFlowing.value =
  false;

isManualFlowAnimationRunning.value =
  false;

isCentralSimulationRunning.value =
  false;

isFlowManuallyPaused.value =
  true;

  const startCount =
    route.directionStarts.length;

  const endCount =
    route.directionEnds.length;

  flowMessage.value =
  "Sentido guardado com " +
  startCount +
  (
    startCount === 1
      ? " início e "
      : " inícios e "
  ) +
  endCount +
  (
    endCount === 1
      ? " fim. "
      : " fins. "
  ) +
  compatiblePaths.length +
  " ligação(ões) compatível(eis) aplicada(s)." +
  (
    rejectedPaths.length > 0
      ? " " +
        rejectedPaths.length +
        " ligação(ões) ambígua(s) foram ignoradas."
      : ""
  ) +
  (
    unconnectedCombinationCount > 0
      ? " " +
        unconnectedCombinationCount +
        " combinação(ões) não tinham ligação."
      : ""
  );
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

async function getPipeGeometryAxis(
  node: FlowNode,
): Promise<PipeGeometryAxis | null> {
  const model =
    loadedModels.get(node.modelId);

  if (!model) {
    return null;
  }

  try {
    const geometryGroups =
      await model.getItemsGeometry([
        node.localId,
      ]);

    const meshDataList =
      geometryGroups[0] ?? [];

    const points: any[] = [];

    for (const meshData of meshDataList) {
      const positions =
        meshData.positions;

      if (
        !positions ||
        positions.length < 3
      ) {
        continue;
      }

            const transform =
        new THREE.Matrix4();

      const rawTransform =
        meshData.transform as any;

      if (
        rawTransform instanceof
        THREE.Matrix4
      ) {
        transform.copy(
          rawTransform,
        );
      } else if (
        rawTransform?.elements &&
        rawTransform.elements.length === 16
      ) {
        transform.fromArray(
          Array.from(
            rawTransform.elements,
          ) as number[],
        );
      } else if (
        Array.isArray(rawTransform) &&
        rawTransform.length === 16
      ) {
        transform.fromArray(
          rawTransform,
        );
      }

      for (
        let positionIndex = 0;
        positionIndex <
        positions.length;
        positionIndex += 3
      ) {
        const point = new THREE.Vector3(
          positions[positionIndex],
          positions[positionIndex + 1],
          positions[positionIndex + 2],
        );

        point.applyMatrix4(transform);

        points.push(point);
      }
    }

    if (points.length < 2) {
      return null;
    }

    const center =
      new THREE.Vector3();

    for (const point of points) {
      center.add(point);
    }

    center.divideScalar(
      points.length,
    );

    let covarianceXX = 0;
    let covarianceXY = 0;
    let covarianceXZ = 0;
    let covarianceYY = 0;
    let covarianceYZ = 0;
    let covarianceZZ = 0;

    for (const point of points) {
      const offsetX =
        point.x - center.x;

      const offsetY =
        point.y - center.y;

      const offsetZ =
        point.z - center.z;

      covarianceXX +=
        offsetX * offsetX;

      covarianceXY +=
        offsetX * offsetY;

      covarianceXZ +=
        offsetX * offsetZ;

      covarianceYY +=
        offsetY * offsetY;

      covarianceYZ +=
        offsetY * offsetZ;

      covarianceZZ +=
        offsetZ * offsetZ;
    }

    let axis = new THREE.Vector3(
      1,
      1,
      1,
    ).normalize();

    for (
      let iteration = 0;
      iteration < 20;
      iteration++
    ) {
      const nextAxis =
        new THREE.Vector3(
          covarianceXX * axis.x +
            covarianceXY * axis.y +
            covarianceXZ * axis.z,

          covarianceXY * axis.x +
            covarianceYY * axis.y +
            covarianceYZ * axis.z,

          covarianceXZ * axis.x +
            covarianceYZ * axis.y +
            covarianceZZ * axis.z,
        );

      if (
        nextAxis.lengthSq() <
        0.000001
      ) {
        return null;
      }

      axis =
        nextAxis.normalize();
    }

    let minimumProjection =
      Number.POSITIVE_INFINITY;

    let maximumProjection =
      Number.NEGATIVE_INFINITY;

    for (const point of points) {
      const projection = point
        .clone()
        .sub(center)
        .dot(axis);

      minimumProjection = Math.min(
        minimumProjection,
        projection,
      );

      maximumProjection = Math.max(
        maximumProjection,
        projection,
      );
    }

    const length =
      maximumProjection -
      minimumProjection;

    if (
      !Number.isFinite(length) ||
      length <= 0.001
    ) {
      return null;
    }

    const start = center
      .clone()
      .addScaledVector(
        axis,
        minimumProjection,
      );

    const end = center
      .clone()
      .addScaledVector(
        axis,
        maximumProjection,
      );

    return {
      start,
      end,
      center,
      length,
    };
  } catch (error) {
    console.error(
      "Erro ao calcular o eixo geométrico do tubo:",
      error,
    );

    return null;
  }
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

async function clearPersistentCircuitHighlights() {
  if (!modelHighlighter) {
    return;
  }

  const styleNames = [
    ...persistentCircuitHighlightStyles,
  ];

  for (const styleName of styleNames) {
    await modelHighlighter.clear(
      styleName,
    );
  }

  persistentCircuitHighlightStyles.clear();

  await fragmentManager.core.update(
    true,
  );
}

async function applyPersistentCircuitHighlight(
  styleName: string,
  color: string,
  modelId: string,
  localIds: number[],
) {
  if (
    !modelHighlighter ||
    localIds.length === 0
  ) {
    return;
  }

  const uniqueLocalIds =
    localIds.filter(
      (
        localId,
        index,
        allLocalIds,
      ) =>
        Number.isFinite(
          localId,
        ) &&
        allLocalIds.indexOf(
          localId,
        ) === index,
    );

  if (!uniqueLocalIds.length) {
    return;
  }

  modelHighlighter.styles.set(
    styleName,
    {
      color:
        new THREE.Color(
          color,
        ),

      opacity:
        0.9,

      transparent:
        true,

      renderedFaces:
        FRAGS.RenderedFaces.TWO,
    },
  );

  persistentCircuitHighlightStyles.add(
    styleName,
  );

  const modelIdMap:
    Record<
      string,
      Set<number>
    > = {};

  modelIdMap[modelId] =
    new Set<number>(
      uniqueLocalIds,
    );

  await modelHighlighter.highlightByID(
    styleName,
    modelIdMap,
    false,
  );
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
  geometryAxis?: PipeGeometryAxis | null,
) {
  const size = new THREE.Vector3();
  const fallbackCenter =
    new THREE.Vector3();

  box.getSize(size);
  box.getCenter(fallbackCenter);

  let start: any;
  let end: any;

  if (geometryAxis) {
    const geometryDirection =
      choosePipeDirection(
        geometryAxis.start.clone(),
        geometryAxis.end.clone(),
        hints,
      );

    start = geometryDirection.start;
    end = geometryDirection.end;
  } else {
    const fallbackAxis =
      getLongestAxis(size);

    const fallbackLength = Math.max(
      size.getComponent(
        fallbackAxis,
      ),
      0.1,
    );

    const endpointA =
      fallbackCenter.clone();

    const endpointB =
      fallbackCenter.clone();

    endpointA.setComponent(
      fallbackAxis,
      fallbackCenter.getComponent(
        fallbackAxis,
      ) -
        fallbackLength / 2,
    );

    endpointB.setComponent(
      fallbackAxis,
      fallbackCenter.getComponent(
        fallbackAxis,
      ) +
        fallbackLength / 2,
    );

    const fallbackDirection =
      choosePipeDirection(
        endpointA,
        endpointB,
        hints,
      );

    start = fallbackDirection.start;
    end = fallbackDirection.end;
  }

  if (
    node &&
    isPipeDirectionReversed(
      node.modelId,
      node.localId,
    )
  ) {
    const originalStart = start;

    start = end;
    end = originalStart;
  }

  const direction = end
    .clone()
    .sub(start);

  const length = Math.max(
    direction.length(),
    0.1,
  );

  direction.normalize();

  const crossSectionSizes = [
    size.x,
    size.y,
    size.z,
  ]
    .filter(
      (dimension) =>
        dimension > 0.001,
    )
    .sort(
      (first, second) =>
        first - second,
    );

  const estimatedPipeDiameter =
    crossSectionSizes[0] ?? 0.08;

  const arrowRadius =
    THREE.MathUtils.clamp(
      Math.pow(
        estimatedPipeDiameter,
        0.75,
      ) * 0.28,
      0.025,
      0.18,
    );

  const arrowLength =
    THREE.MathUtils.clamp(
      Math.pow(
        estimatedPipeDiameter,
        0.75,
      ) * 0.9,
      0.07,
      0.48,
    );

  const geometry =
    new THREE.ConeGeometry(
      arrowRadius,
      arrowLength,
      8,
    );

  const material =
    getRouteMaterialForNode(
      temperature,
      node,
    );

  const particleCount = Math.max(
    1,
    Math.round(length / 0.4),
  );

  for (
    let particleIndex = 0;
    particleIndex < particleCount;
    particleIndex++
  ) {
    const mesh = new THREE.Mesh(
      geometry,
      material,
    );

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
      offset:
        particleIndex /
        particleCount,
      length,
    });
  }
}

function choosePipeDirection(
  endpointA: any,
  endpointB: any,
  hints: PipeDirectionHints,
) {
  let start = endpointA;
  let end = endpointB;

  if (hints.upstream && hints.downstream) {
    const optionA =
      endpointA.distanceTo(hints.upstream) +
      endpointB.distanceTo(hints.downstream);

    const optionB =
      endpointB.distanceTo(hints.upstream) +
      endpointA.distanceTo(hints.downstream);

    if (optionA <= optionB) {
      start = endpointA;
      end = endpointB;
    } else {
      start = endpointB;
      end = endpointA;
    }

    return { start, end };
  }

  if (hints.upstream) {
    start = closestEndpoint(
      endpointA,
      endpointB,
      hints.upstream,
    );

    end =
      start === endpointA
        ? endpointB
        : endpointA;

    return { start, end };
  }

  if (hints.downstream) {
    end = closestEndpoint(
      endpointA,
      endpointB,
      hints.downstream,
    );

    start =
      end === endpointA
        ? endpointB
        : endpointA;
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

function clearAutomaticAnalysisResults() {
  hasAutomaticAnalysisResults.value = false;
  isAutomaticAnalysisRunning.value = false;

  automaticAnalysisResults.pipes = 0;
  automaticAnalysisResults.valves = 0;
  automaticAnalysisResults.equipment = 0;
  automaticAnalysisResults.total = 0;

  flowMessage.value =
    "Resultados da análise automática limpos. A configuração manual foi mantida.";
}

async function startAutomaticAnalysis() {
  if (!loadedModels.size) {
    flowMessage.value =
      "Carrega primeiro um ficheiro IFC no painel da esquerda.";
    return;
  }

  isAutomaticAnalysisRunning.value = true;
  hasAutomaticAnalysisResults.value = false;

  automaticAnalysisResults.pipes = 0;
  automaticAnalysisResults.valves = 0;
  automaticAnalysisResults.equipment = 0;
  automaticAnalysisResults.total = 0;

  try {
    for (const model of loadedModels.values()) {
      const pipeCategories = await model.getItemsOfCategories([
        /IFCPIPESEGMENT/i,
        /IFCFLOWSEGMENT/i,
        /IFCPIPEFITTING/i,
        /IFCFLOWFITTING/i,
      ]);

      const valveCategories = await model.getItemsOfCategories([
        /IFCVALVE/i,
        /IFCFLOWCONTROLLER/i,
      ]);

      const equipmentCategories = await model.getItemsOfCategories([
        /IFCPUMP/i,
        /IFCBOILER/i,
        /IFCTANK/i,
        /IFCHEATEXCHANGER/i,
        /IFCFLOWSTORAGEDEVICE/i,
        /IFCFLOWMOVINGDEVICE/i,
      ]);

      const pipeIds = new Set(
        Object.values(pipeCategories).flat(),
      );

      const valveIds = new Set(
        Object.values(valveCategories).flat(),
      );

      const equipmentIds = new Set(
        Object.values(equipmentCategories).flat(),
      );

      automaticAnalysisResults.pipes += pipeIds.size;
      automaticAnalysisResults.valves += valveIds.size;
      automaticAnalysisResults.equipment += equipmentIds.size;
    }

    automaticAnalysisResults.total =
      automaticAnalysisResults.pipes +
      automaticAnalysisResults.valves +
      automaticAnalysisResults.equipment;

    hasAutomaticAnalysisResults.value = true;

    flowMessage.value =
      "Análise automática inicial concluída.";
  } catch (error) {
    console.error(
      "Erro durante a análise automática:",
      error,
    );

    flowMessage.value =
      "Não foi possível concluir a análise automática.";
  } finally {
    isAutomaticAnalysisRunning.value = false;
  }
}

function getAutomaticCircuitKindFromText(
  value: string,
): CycleCircuitKind {
  const normalizedValue = String(
    value ?? "",
  )
    .trim()
    .toLowerCase()
    .normalize("NFD")
    .replace(
      /[\u0300-\u036f]/g,
      "",
    )
    .replace(/[_-]/g, " ")
    .replace(/\s+/g, " ");

  const normalizedWords =
    normalizedValue.split(" ");

  const hasReturn =
    normalizedValue.includes(
      "retorno",
    ) ||
    normalizedValue.includes(
      "return",
    ) ||
    normalizedValue.includes(
      "recirculacao",
    ) ||
    normalizedValue.includes(
      "recirculation",
    ) ||
    normalizedWords.includes(
      "ret",
    );

  const hasCold =
    normalizedValue.includes(
      "agua fria",
    ) ||
    normalizedValue.includes(
      "cold water",
    ) ||
    normalizedValue.includes(
      "domestic cold water",
    ) ||
    normalizedWords.includes(
      "fria",
    ) ||
    normalizedWords.includes(
      "frio",
    ) ||
    normalizedWords.includes(
      "cold",
    ) ||
    normalizedWords.includes(
      "afs",
    ) ||
    normalizedWords.includes(
      "af",
    ) ||
    normalizedWords.includes(
      "dcw",
    );

  const hasHot =
    normalizedValue.includes(
      "agua quente",
    ) ||
    normalizedValue.includes(
      "hot water",
    ) ||
    normalizedValue.includes(
      "domestic hot water",
    ) ||
    normalizedWords.includes(
      "quente",
    ) ||
    normalizedWords.includes(
      "hot",
    ) ||
    normalizedWords.includes(
      "aqs",
    ) ||
    normalizedWords.includes(
      "acs",
    ) ||
    normalizedWords.includes(
      "dhw",
    );

  const hasSupply =
    normalizedWords.includes(
      "avanco",
    ) ||
    normalizedWords.includes(
      "ida",
    ) ||
    normalizedWords.includes(
      "supply",
    );

  if (hasCold && hasReturn) {
    return "coldReturn";
  }

  if (hasCold) {
    return "coldSupply";
  }

  if (hasHot && hasReturn) {
    return "hotReturn";
  }

  if (hasHot) {
    return "hotSupply";
  }

  if (hasReturn) {
    return "hotReturn";
  }

  if (hasSupply) {
    return "hotSupply";
  }

  return "extra";
}

function getAutomaticCircuitKindFromSystemGroup(
  systemName: string,
  systemTypes: Set<string>,
): CycleCircuitKind {
  const detectedKinds = new Set<CycleCircuitKind>();

  for (const systemType of systemTypes) {
    const kind =
      getAutomaticCircuitKindFromText(systemType);

    if (kind !== "extra") {
      detectedKinds.add(kind);
    }
  }

  if (detectedKinds.size === 1) {
    return [...detectedKinds][0];
  }

  if (detectedKinds.size > 1) {
    return "extra";
  }

  return getAutomaticCircuitKindFromText(systemName);
}

async function createAutomaticCircuitsFromSystemNames() {
  if (!systemScanResults.systemNames.length) {
    flowMessage.value =
      "Faz primeiro a análise dos sistemas IFC.";
    return;
  }

  let createdCount = 0;
  let existingCount = 0;
  let assignedPipeCount = 0;
  let unidentifiedCount = 0;
  flowConnections.splice(0);
  for (
  const [modelId, ignoredIds] of
    ignoredAutomaticPathNodes
) {
  for (const localId of ignoredIds) {
    for (
      const circuitKey of
        getAllKnownCircuitKeys()
    ) {
      getAssignmentSet(
        circuitKey,
        modelId,
      ).delete(localId);
    }

    hiddenFlowArrowElements
      .get(modelId)
      ?.delete(localId);

    reversedPipeDirections
      .get(modelId)
      ?.delete(localId);

    syncedPipeDirections
      .get(modelId)
      ?.delete(localId);
  }
}
  automaticDirectionNeighbors.clear();
automaticOrderedCircuitNodes.clear();

  for (
    const systemName of systemScanResults.systemNames
  ) {
    const systemGroup =
      scannedSystemGroups.get(systemName);

    if (!systemGroup) {
      continue;
    }

    const normalizedSystemName =
      systemName.trim().toLowerCase();

    let circuit =
      cycleCircuitDefinitions.find(
        (existingCircuit) =>
          existingCircuit.name
            .trim()
            .toLowerCase() ===
          normalizedSystemName,
      );

    const detectedKind =
  getAutomaticCircuitKindFromSystemGroup(
    systemName,
    systemGroup.systemTypes,
  );

if (circuit) {
  existingCount++;

  if (
    detectedKind !== "extra" &&
    circuit.kind !== detectedKind
  ) {
    const automaticColor =
      getNextCircuitColorForKind(detectedKind);

    circuit.kind = detectedKind;
    circuit.color = automaticColor;
    circuit.defaultColor = automaticColor;
  }

  if (detectedKind === "extra") {
    unidentifiedCount++;
  }
} else {
  const kind = detectedKind;

  if (kind === "extra") {
    unidentifiedCount++;
  }

  const color =
    getNextCircuitColorForKind(kind);

      const key =
        "cycle" +
        activeCycleNumber.value +
        "-" +
        kind +
        "-" +
        crypto.randomUUID();

      circuit = {
        key,
        cycleNumber: activeCycleNumber.value,
        kind,
        name: systemName,
        color,
        defaultColor: color,
      };

      cycleCircuitDefinitions.push(circuit);
      createdCount++;
    }

    if (!manualAssignments[circuit.key]) {
      manualAssignments[circuit.key] =
        new Map();
    }

    const orderedNodes = [
      ...systemGroup.nodes,
    ];

    await updateAutomaticDirectionNeighbors(
      circuit.key,
      orderedNodes,
    );

automaticOrderedCircuitNodes.set(
  circuit.key,
  orderedNodes.map((node) => ({
    modelId: node.modelId,
    localId: node.localId,
  })),
);
for (const node of orderedNodes) {
  reversedPipeDirections
  .get(node.modelId)
  ?.delete(node.localId);

syncedPipeDirections
  .get(node.modelId)
  ?.delete(node.localId);

  for (
    const existingCircuitKey of
      getAllKnownCircuitKeys()
  ) {
    getAssignmentSet(
      existingCircuitKey,
      node.modelId,
    ).delete(node.localId);
  }

  getAssignmentSet(
    circuit.key,
    node.modelId,
  ).add(node.localId);

  assignedPipeCount++;
}

for (
  let nodeIndex = 0;
  nodeIndex < orderedNodes.length - 1;
  nodeIndex++
) {
  const fromNode = orderedNodes[nodeIndex];
  const toNode = orderedNodes[nodeIndex + 1];

  const connection: FlowConnection = {
    from: {
      modelId: fromNode.modelId,
      localId: fromNode.localId,
    },
    to: {
      modelId: toNode.modelId,
      localId: toNode.localId,
    },
    temperature: circuit.key,
  };

  addFlowConnectionIfMissing(connection);
}
  }

  saveCycleCircuitDefinitionsToStorage();
  saveReversedDirectionsToStorage();
saveSyncedPipeDirectionsToStorage();
selectDefaultCircuitForActiveCycle();
updateManualStats();

circuitMaterialCache.clear();

await rebuildManualFlowLayer();

  flowMessage.value =
    createdCount +
    " caminho(s) criado(s). " +
    existingCount +
    " já existiam. " +
    assignedPipeCount +
    " tubo(s) associados. " +
    unidentifiedCount +
    " caminho(s) precisam de revisão manual.";
}

function isIgnoredAutomaticPathElement(data: any) {
  const elementText = flattenItemText(data)
    .toLowerCase()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .replace(/[_-]/g, " ");

  return (
    elementText.includes("rigid coupling") ||
    elementText.includes("acoplamento rigido") ||
    elementText.includes("uniao rigida") ||
    elementText.includes("flange") ||
    elementText.includes("flanged")
  );
}

function isShutoffValveIfcElement(
  rawIfcData: any,
) {
  const revitValues =
    collectRevitFamilyAndTypeValues(
      rawIfcData,
    );

  const familyFromProperty =
    findIfcPropertyValue(
      rawIfcData,
      [
        "Family",
        "Família",
        "Familia",
        "Revit Family",
        "RevitFamily",
        "Family Name",
        "FamilyName",
      ],
    );

  const collectedFamily = String(
    revitValues.family ?? "",
  ).trim();

  const propertyFamily = String(
    familyFromProperty ?? "",
  ).trim();

  const familyValue =
    collectedFamily &&
    collectedFamily !==
      "não encontrado"
      ? collectedFamily
      : propertyFamily !==
          "não encontrado"
        ? propertyFamily
        : "";

  const normalizedFamily =
    familyValue
      .toLowerCase()
      .normalize("NFD")
      .replace(
        /[\u0300-\u036f]/g,
        "",
      )
      .replace(/[_-]/g, " ")
      .replace(/\s+/g, " ")
      .trim();

  const isValveFamily =
    normalizedFamily.includes(
      "valvula",
    ) ||
    normalizedFamily.includes(
      "valve",
    );

  return {
    isShutoffValve:
      isValveFamily,

    familyValue,

    normalizedFamily,
  };
}

async function scanIfcValveStates() {
  let detectedValveCount = 0;
  let normallyClosedCount = 0;
  let normallyOpenCount = 0;

  for (const model of loadedModels.values()) {
    const valveCategories =
  await model.getItemsOfCategories([
    /IFCVALVE/i,
    /IFCFLOWCONTROLLER/i,
    /IFCFLOWFITTING/i,
    /IFCPIPEFITTING/i,
    /IFCFLOWTERMINAL/i,
  ]);

    const valveLocalIds = [
      ...new Set(
        Object.values(
          valveCategories,
        ).flat(),
      ),
    ];

    for (
      let startIndex = 0;
      startIndex < valveLocalIds.length;
      startIndex += 100
    ) {
      const currentIds =
        valveLocalIds.slice(
          startIndex,
          startIndex + 100,
        );

const itemsData =
  await model.getItemsData(
    currentIds,
    {
      attributesDefault: true,

      relations: {
        IsTypedBy: {
          attributes: true,
          relations: true,
        },

        IsDefinedBy: {
          attributes: true,
          relations: true,
        },

        DefinesOccurrence: {
          attributes: true,
          relations: true,
        },

        HasAssignments: {
          attributes: true,
          relations: false,
        },
      },
    },
  );

      for (
        let itemIndex = 0;
        itemIndex < itemsData.length;
        itemIndex++
      ) {
        const itemData =
          itemsData[itemIndex];

        const localId =
          currentIds[itemIndex];

        if (
          !itemData ||
          !Number.isFinite(localId)
        ) {
          continue;
        }

        const rawIfcData =
  normalizeIfcValue(itemData);

const valveIdentification =
  isShutoffValveIfcElement(
    rawIfcData,
  );

const key = elementKey(
  model.modelId,
  localId,
);

const existingElement =
  mepElements[key];

if (
  !valveIdentification
    .isShutoffValve
) {
  if (
    existingElement &&
    isValveElementType(
      existingElement.elementType,
    )
  ) {
    delete mepElements[key];
  }

  continue;
}

const stateValue =
  findIfcPropertyValue(
    rawIfcData,
    [
      "State",
      "Estado",
      "Valve State",
      "ValveState",
    ],
  );

        const mechanicalSystemName =
  findMechanicalSystemName(
    rawIfcData,
  );

const fallbackSystemName =
  findIfcPropertyValue(
    rawIfcData,
    [
      "System Name",
      "SystemName",
      "System Abbreviation",
      "SystemAbbreviation",
    ],
  );

const systemName =
  mechanicalSystemName ||
  (
    fallbackSystemName !==
      "não encontrado"
      ? fallbackSystemName
      : ""
  );

const systemType =
  findIfcPropertyValue(
    rawIfcData,
    [
      "System Type",
      "SystemType",
      "System Classification",
      "SystemClassification",
    ],
  );

        const valveType =
          getValveTypeFromIfcState(
            stateValue,
          );

        mepElements[key] = {
  ...existingElement,

  modelId: model.modelId,
  localId,

  elementType: valveType,

  circuitType:
    existingElement
      ?.circuitType ??
    "unknown",

  systemName:
    systemName ||
    existingElement?.systemName ||
    "",

  systemType:
    systemType !== "não encontrado"
      ? systemType
      : existingElement?.systemType ||
        "",

  ifcNormalState:
  stateValue !== "não encontrado"
    ? stateValue
    : "",

isShutoffValve:
  valveIdentification.isShutoffValve,

valveIdentificationText:
  valveIdentification.familyValue,

state:
  existingElement?.state ===
    "closed"
    ? "closed"
    : "open",
};

        detectedValveCount++;

        if (
          valveType ===
          "normallyClosedValve"
        ) {
          normallyClosedCount++;
        } else {
          normallyOpenCount++;
        }
      }
    }
  }

  saveMepElementsToStorage();

  return {
    detectedValveCount,
    normallyClosedCount,
    normallyOpenCount,
  };
}

async function scanIfcSystems() {
  if (!loadedModels.size) {
    flowMessage.value =
      "Carrega primeiro um ficheiro IFC.";
    return;
  }

  isSystemScanRunning.value = true;
    excludedValveKeys.clear();
  selectedValveKeysForManagement.clear();

  await clearManagedValveHighlight();

  removeActiveIfcStorageItem(
    EXCLUDED_VALVES_STORAGE_KEY,
  );
  hasSystemScanResults.value = false;

  isSystemTypesListOpen.value = false;
  isSystemNamesListOpen.value = false;

  systemScanResults.systemTypes.splice(0);
  systemScanResults.systemNames.splice(0);

  const uniqueSystemTypes = new Set<string>();
  const uniqueSystemNames = new Set<string>();

  scannedSystemGroups.clear();
  pipeTypeFlowNodes.clear();
  ignoredAutomaticPathNodes.clear();

  try {
        const detectedValves =
      await scanIfcValveStates();

    for (const model of loadedModels.values()) {

      const categories =
        await model.getItemsOfCategories([
          /IFCPIPESEGMENT/i,
          /IFCFLOWSEGMENT/i,
          /IFCPIPEFITTING/i,
          /IFCFLOWFITTING/i,
        ]);

      const localIds = [
        ...new Set(
          Object.values(categories).flat(),
        ),
      ];

      for (
        let startIndex = 0;
        startIndex < localIds.length;
        startIndex += 100
      ) {
        const currentIds = localIds.slice(
          startIndex,
          startIndex + 100,
        );

        const itemsData =
          await model.getItemsData(
            currentIds,
            {
              attributesDefault: true,
              relations: {
                IsDefinedBy: {
                  attributes: true,
                  relations: true,
                },
                HasAssignments: {
                  attributes: true,
                  relations: false,
                },
              },
            },
          );

        for (
          let itemIndex = 0;
          itemIndex < itemsData.length;
          itemIndex++
        ) {
          const itemData = itemsData[itemIndex];
          const localId = currentIds[itemIndex];

          if (
            !itemData ||
            !Number.isFinite(localId)
          ) {
            continue;
          }

          const rawIfcData =
            normalizeIfcValue(itemData);

          const revitValues =
            collectRevitFamilyAndTypeValues(
              rawIfcData,
            );

          const normalizedFamily =
            String(
              revitValues.family ?? "",
            )
              .trim()
              .toLowerCase()
              .normalize("NFD")
              .replace(
                /[\u0300-\u036f]/g,
                "",
              )
              .replace(/[_-]/g, " ")
              .replace(/\s+/g, " ");

          if (
            normalizedFamily ===
            "pipe types"
          ) {
            pipeTypeFlowNodes.add(
              nodeKey({
                modelId:
                  model.modelId,
                localId,
              }),
            );
          }
          
          if (isIgnoredAutomaticPathElement(rawIfcData)) {
  const ignoredIds =
    ignoredAutomaticPathNodes.get(
      model.modelId,
    ) ?? new Set<number>();

  ignoredIds.add(localId);

  ignoredAutomaticPathNodes.set(
    model.modelId,
    ignoredIds,
  );

  continue;
}

          const revitElementIdText =
  getAttributeValueText(
    itemData?.Tag ??
    itemData?.tag,
  ).trim() ||
  findIfcPropertyValue(
    rawIfcData,
    [
      "Tag",
      "BATID",
      "Element ID",
      "ElementId",
    ],
  );

const revitElementId =
  Number(
    String(revitElementIdText)
      .replace(/[^\d]/g, ""),
  );

          const systemType =
            findIfcPropertyValue(
              rawIfcData,
              [
                "System Type",
                "SystemType",
                "System Classification",
                "SystemClassification",
              ],
            );

          const mechanicalSystemName =
            findMechanicalSystemName(
              rawIfcData,
            );

          const fallbackSystemName =
            findIfcPropertyValue(
              rawIfcData,
              [
                "System Name",
                "SystemName",
              ],
            );

          const systemName =
            mechanicalSystemName ||
            fallbackSystemName;

          const hasSystemType =
            systemType &&
            systemType !== "não encontrado";

          const hasSystemName =
            systemName &&
            systemName !== "não encontrado";

          if (hasSystemType) {
            uniqueSystemTypes.add(
              String(systemType).trim(),
            );
          }

          if (!hasSystemName) {
            continue;
          }

          const cleanSystemName =
            String(systemName).trim();

          uniqueSystemNames.add(cleanSystemName);

          const systemGroup =
            scannedSystemGroups.get(
              cleanSystemName,
            ) ?? {
              nodes: [],
              systemTypes: new Set<string>(),
            };

          const nodeAlreadyExists =
            systemGroup.nodes.some(
              (node) =>
                node.modelId === model.modelId &&
                node.localId === localId,
            );

          if (!nodeAlreadyExists) {
  systemGroup.nodes.push({
    modelId: model.modelId,
    localId,
    revitElementId:
      Number.isFinite(revitElementId)
        ? revitElementId
        : Number.MAX_SAFE_INTEGER,
  });
}

          if (hasSystemType) {
            systemGroup.systemTypes.add(
              String(systemType).trim(),
            );
          }

          scannedSystemGroups.set(
            cleanSystemName,
            systemGroup,
          );
        }
      }
    }

    systemScanResults.systemTypes.splice(
      0,
      systemScanResults.systemTypes.length,
      ...[...uniqueSystemTypes].sort(
        (first, second) =>
          first.localeCompare(second),
      ),
    );

    systemScanResults.systemNames.splice(
      0,
      systemScanResults.systemNames.length,
      ...[...uniqueSystemNames].sort(
        (first, second) =>
          first.localeCompare(second),
      ),
    );

    savePipeTypeFlowNodesToStorage();

    hasSystemScanResults.value = true;

    flowMessage.value =
  "Scan dos sistemas IFC concluído. " +
  detectedValves.detectedValveCount +
  " válvula(s) reconhecida(s): " +
  detectedValves.normallyClosedCount +
  " normalmente fechada(s) e " +
  detectedValves.normallyOpenCount +
  " normalmente aberta(s).";
    } catch (error) {
    console.error(
      "Erro ao analisar os sistemas IFC:",
      error,
    );

    const errorMessage =
      error instanceof Error
        ? error.message
        : String(error);

    flowMessage.value =
      "Não foi possível analisar os sistemas IFC: " +
      errorMessage;

    window.alert(
      "Erro durante o scan do IFC:\n\n" +
      errorMessage,
    );
  } finally {
    isSystemScanRunning.value = false;
  }
}

function toggleElementPanelMinimized() {
  isElementPanelMinimized.value = !isElementPanelMinimized.value;
}

function toggleIfcInformationPanelMinimized() {
  isIfcInformationPanelMinimized.value =
    !isIfcInformationPanelMinimized.value;
}

function toggleFlowControlsPanelMinimized() {
  isFlowControlsPanelMinimized.value = !isFlowControlsPanelMinimized.value;
}

async function applySimulationCycleFilter() {
  if (!loadedModels.size) {
    flowMessage.value =
      "Carrega primeiro um ficheiro IFC.";

    return;
  }

  if (!selectedSimulationCycles.size) {
    flowMessage.value =
      "Seleciona pelo menos um ciclo para visualizar.";

    return;
  }

  if (
    highlightedSavedRouteId.value &&
    modelHighlighter
  ) {
    await modelHighlighter.clear(
      "saved-route-highlight",
    );

    highlightedSavedRouteId.value = null;
  }

  isCycleViewFilterActive.value = true;

  await rebuildManualFlowLayer();

  flowMessage.value =
    "A visualizar os ciclos: " +
    getSelectedSimulationCyclesLabel() +
    ".";
}

function toggleResetPanelMinimized() {
  isResetPanelMinimized.value =
    !isResetPanelMinimized.value;
}

function toggleSimulationControlPanelMinimized() {
  isSimulationControlPanelMinimized.value =
    !isSimulationControlPanelMinimized.value;
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

function updateSelectedTubeRouteInfo() {
  const selectedNode =
    getFirstSelectedNode();

  if (!selectedNode) {
    selectedTubeRouteInfo.value = "";

    return;
  }

  const routesWithSelectedNode =
    savedRoutes.filter(
      (route) =>
        routeContainsAdaptedNode(
          route,
          selectedNode,
        ),
    );

  if (
    routesWithSelectedNode.length === 0
  ) {
    selectedTubeRouteInfo.value =
      "O elemento selecionado não pertence a nenhum percurso guardado.";

    return;
  }

  const routeNames = [
    ...new Set(
      routesWithSelectedNode.map(
        (route) =>
          route.name,
      ),
    ),
  ];

  selectedTubeRouteInfo.value =
    routeNames.length === 1
      ? routeNames[0]
      : routeNames.length +
        " percursos: " +
        routeNames.join(" · ");
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

async function updateFlowAnimationWithoutStarting() {
  if (
    selectedValveDesignationKey.value ||
    isValveFocusModeActive.value ||
    highlightedValveFromDropdown.value
  ) {
    const valveWasCleared =
      await clearSelectedValveFromDropdown(
        false,
      );

    if (!valveWasCleared) {
      flowMessage.value =
        "Atualização cancelada porque existem alterações não guardadas na válvula.";

      return;
    }
  }

  isFlowing.value =
    false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value = true;

  flowMessage.value =
    "A atualizar a animação.";

  await rebuildManualFlowLayer();

  if (
    hasFlowPreparationError.value ||
    !pipeParticles.length ||
    !isFlowAnimationReady.value
  ) {
    flowMessage.value =
      "Não foi possível preparar a animação.";

    return;
  }

  isFlowing.value = false;

  isManualFlowAnimationRunning.value =
    false;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value = true;

  flowMessage.value =
    "Animação pronta. Clica em Iniciar simulação.";
}

async function toggleFlow() {
    if (highlightedSavedRouteId.value) {
    flowMessage.value =
      "Limpa primeiro o realce do caminho antes de iniciar a simulação.";

    return;
  }

  if (isPreparingFlowAnimation.value) {
    flowMessage.value =
      "Aguarda até a preparação da animação terminar.";

    return;
  }

  if (isFlowing.value) {
    isFlowing.value = false;

    isManualFlowAnimationRunning.value =
      false;

    isCentralSimulationRunning.value =
      false;

    isFlowManuallyPaused.value = true;

    flowMessage.value =
      "Simulação pausada.";

    return;
  }

    if (
    selectedRouteIdsForSimulation.size > 0
  ) {
    selectedRouteIdsForSimulation.clear();

    flowMessage.value =
      "A preparar a simulação geral.";

    await rebuildManualFlowLayer();

    if (
      hasFlowPreparationError.value ||
      !pipeParticles.length ||
      !isFlowAnimationReady.value
    ) {
      flowMessage.value =
        "Não foi possível preparar a simulação geral.";

      return;
    }
  }

  if (
    !isFlowAnimationReady.value ||
    !pipeParticles.length
  ) {
    flowMessage.value =
      "Clica primeiro em Atualizar e aguarda até a animação estar pronta.";

    return;
  }

  if (
    hasFlowPreparationError.value ||
    !pipeParticles.length
  ) {
    flowMessage.value =
      "Não existem setas disponíveis para iniciar a simulação.";

    return;
  }

  isFlowing.value = true;

  isManualFlowAnimationRunning.value =
    true;

  isCentralSimulationRunning.value =
    false;

  isFlowManuallyPaused.value = false;

  flowMessage.value =
    "Simulação iniciada.";
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

async function clearCurrentModelSelection() {
  selectedItems.clear();

  selectedCount.value =
    0;

  selectedIfcDetailsText.value =
    "";

  isIfcDetailsPanelOpen.value =
    false;

  selectedMepElementInfo.value =
    "Nenhum elemento classificado selecionado.";

  selectedTubeRouteInfo.value =
    "";

  if (modelHighlighter) {
    await modelHighlighter.clear(
      "select",
    );
  }

  await fragmentManager.core.update(
    true,
  );

  flowMessage.value =
    "Seleção atual limpa.";
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
  top: 60px;
  z-index: 1000;
  width: min(390px, calc(100vw - 48px));
  max-height: calc(100vh - 84px);
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
  width: 100%;
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

.flow-actions button:disabled {
  background: #6f7376;
  color: #4c4f51;
  cursor: not-allowed;
  opacity: 0.75;
}

.flow-actions button:disabled:hover {
  background: #6f7376;
}

.selected-pipe-arrow-actions button:disabled {
  background: #6f7376;
  color: #4c4f51;
  cursor: not-allowed;
  opacity: 0.75;
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
  display: grid;
  grid-template-columns:
    minmax(0, 1fr);
  align-items: stretch;
  gap: 8px;
  margin-top: 6px;
  padding: 8px;
  border-radius: 6px;
  background:
    rgba(255, 255, 255, 0.08);
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
  border: 1px solid
    rgba(255, 193, 7, 0.72);
  background:
    rgba(255, 193, 7, 0.14);
  box-shadow:
    inset 0.22rem 0 0
    rgba(255, 193, 7, 0.9);
}

.saved-route-item--locked strong {
  color: #ffd166;
}

.saved-route-item--locked.saved-route-item--highlighted {
  border: 1px solid
    rgba(0, 229, 255, 0.85);
  background:
    rgba(0, 229, 255, 0.14);
  box-shadow:
    inset 0.22rem 0 0
    rgba(0, 229, 255, 0.95);
}

.saved-route-item--locked.saved-route-item--highlighted strong {
  color: #00e5ff;
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
  position: relative;
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
  top: 60px;
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

.ifc-details-output {
  max-height: 320px;
  overflow: auto;
  margin-top: 10px;
  padding: 10px;
  border-radius: 6px;
  background: rgba(0, 0, 0, 0.42);
  color: #dbe9f1;
  font-size: 0.68rem;
  line-height: 1.35;
  white-space: pre-wrap;
  word-break: break-word;
}

.application-tabs {
  position: fixed;
  top: 0;
  left: 23rem;
  right: 0;
  z-index: 1005;
  display: flex;
  align-items: flex-end;
  gap: 4px;
  height: 46px;
  padding: 6px 14px 0;
  background: rgba(13, 22, 28, 0.96);
  border-bottom: 1px solid rgba(255, 255, 255, 0.18);
}

.application-tabs--ifc-collapsed {
  left: 0;
}

.application-tab {
  min-height: 36px;
  padding: 7px 18px;
  border: 1px solid transparent;
  border-radius: 8px 8px 0 0;
  background: rgba(255, 255, 255, 0.08);
  color: #b8c9d3;
  cursor: pointer;
  font-size: 0.76rem;
  font-weight: 800;
}

.application-tab:hover {
  background: rgba(143, 211, 255, 0.14);
  color: #f7fbff;
}

.application-tab--active {
  border-color: rgba(143, 211, 255, 0.45);
  border-bottom-color: #1d2932;
  background: #1d2932;
  color: #8fd3ff;
}

.automatic-analysis-status {
  margin: 12px 0 0;
  padding: 10px;
  border-radius: 6px;
  font-size: 0.8rem;
  font-weight: 800;
  line-height: 1.35;
}

.automatic-analysis-status--ready {
  border: 1px solid rgba(102, 187, 106, 0.55);
  background: rgba(102, 187, 106, 0.14);
  color: #a5d6a7;
}

.automatic-analysis-results {
  display: grid;
  gap: 8px;
  margin-top: 14px;
}

.automatic-analysis-result {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 9px 10px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.08);
  color: #dbe9f1;
  font-size: 0.78rem;
  font-weight: 800;
}

.automatic-analysis-result strong {
  color: #8fd3ff;
  font-size: 0.95rem;
}

.automatic-analysis-result--total {
  border: 1px solid rgba(143, 211, 255, 0.45);
  background: rgba(143, 211, 255, 0.14);
}

.automatic-system-result {
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.06);
}

.automatic-system-result__header {
  display: grid;
  grid-template-columns: 1fr auto auto;
  align-items: center;
  gap: 10px;
  padding: 10px;
  color: #dbe9f1;
  font-size: 0.78rem;
  font-weight: 800;
}

.automatic-system-result__header strong {
  color: #8fd3ff;
  font-size: 0.95rem;
}

.automatic-system-result__list {
  display: grid;
  gap: 5px;
  max-height: 240px;
  padding: 0 10px 10px;
  overflow-y: auto;
}

.automatic-system-result__list span {
  padding: 7px 8px;
  border-radius: 4px;
  background: rgba(143, 211, 255, 0.1);
  color: #dbe9f1;
  font-size: 0.74rem;
  font-weight: 700;
  overflow-wrap: anywhere;
}

.automatic-system-result__list
  .automatic-system-result__empty {
  color: #9fb0ba;
  font-style: italic;
  font-weight: 600;
}

.saved-route-group-actions {
  display: grid;
  gap: 7px;
  margin: 8px 0;
  padding: 8px;
  border-radius: 6px;
  background: rgba(143, 211, 255, 0.1);
}

.saved-route-search {
  display: grid;
  gap: 0.4rem;
  width: 100%;
  min-width: 0;
}

.saved-route-search__control {
  display: flex;
  width: 100%;
  min-width: 0;
}

.saved-route-search__control input {
  display: block;
  width: 100%;
  min-width: 0;
  box-sizing: border-box;
}

.saved-route-group-actions span {
  color: #dbe9f1;
  font-size: 0.72rem;
  font-weight: 800;
}

.saved-route-group-checkbox {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  color: #dbe9f1;
  font-size: 0.68rem;
  font-weight: 700;
  cursor: pointer;
}

.saved-route-group-checkbox input {
  margin: 0;
  accent-color: #8fd3ff;
}

.saved-route-group-color {
  display: inline-block;
  width: 11px;
  height: 11px;
  margin-right: 6px;
  border: 1px solid rgba(
    255,
    255,
    255,
    0.55
  );
  border-radius: 50%;
  vertical-align: middle;
}

.saved-route-cycle-selector {
  display: grid;
  gap: 5px;
}

.saved-route-cycle-selector span {
  color: #dbe9f1;
  font-size: 0.72rem;
  font-weight: 800;
}

.saved-route-cycle-selector select {
  width: 100%;
  min-height: 34px;
  padding: 6px 8px;
  border: 1px solid
    rgba(143, 211, 255, 0.35);
  border-radius: 5px;
  background: rgba(7, 19, 26, 0.85);
  color: #dbe9f1;
  font-size: 0.74rem;
  font-weight: 700;
}

.cycle-view-button--active {
  border-color: rgba(
    102,
    187,
    106,
    0.75
  ) !important;
  background: rgba(
    102,
    187,
    106,
    0.22
  ) !important;
  color: #b9f6ca !important;
}

.simulation-cycle-selection {
  display: grid;
  gap: 6px;
  margin: 7px 0;
  padding: 8px;
  border: 1px solid
    rgba(143, 211, 255, 0.25);
  border-radius: 6px;
  background: rgba(7, 19, 26, 0.55);
}

.simulation-cycle-option {
  display: flex;
  align-items: center;
  gap: 7px;
  min-height: 28px;
  color: #dbe9f1;
  font-size: 0.74rem;
  font-weight: 700;
  cursor: pointer;
}

.simulation-cycle-option input {
  margin: 0;
  accent-color: #8fd3ff;
}

.route-group-dialog-backdrop {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: grid;
  place-items: center;
  padding: 20px;
  background: rgba(3, 8, 12, 0.68);
  backdrop-filter: blur(4px);
}

.route-group-dialog {
  width: min(390px, calc(100vw - 40px));
  padding: 16px;
  border: 1px solid rgba(143, 211, 255, 0.4);
  border-radius: 10px;
  background: #17232c;
  color: #f7fbff;
  box-shadow: 0 20px 55px rgba(0, 0, 0, 0.45);
}

.route-group-dialog__header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 16px;
}

.route-group-dialog__header h2 {
  margin: 3px 0 0;
  font-size: 1rem;
}

.route-group-dialog__field {
  display: grid;
  gap: 7px;
  margin-top: 12px;
}

.route-group-dialog__field > span {
  color: #dbe9f1;
  font-size: 0.76rem;
  font-weight: 800;
}

.route-group-dialog__field input[type="text"] {
  width: 100%;
  min-height: 36px;
  padding: 7px 9px;
  border: 1px solid rgba(143, 211, 255, 0.35);
  border-radius: 5px;
  background: rgba(7, 19, 26, 0.85);
  color: #f7fbff;
}

.route-group-color-control {
  display: grid;
  grid-template-columns: 48px 30px 1fr;
  align-items: center;
  gap: 9px;
}

.route-group-color-control input[type="color"] {
  width: 48px;
  height: 36px;
  padding: 2px;
  border: 1px solid rgba(255, 255, 255, 0.28);
  border-radius: 5px;
  background: transparent;
  cursor: pointer;
}

.route-group-color-preview {
  display: block;
  width: 26px;
  height: 26px;
  border: 2px solid #f7fbff;
  border-radius: 50%;
}

.route-group-color-control strong {
  color: #dbe9f1;
  font-size: 0.72rem;
  line-height: 1.3;
}

.saved-route-direction-status span {
  color: #dbe9f1;
  font-size: 0.72rem;
  font-weight: 800;
}

.route-direction-dialog-backdrop {
  position: fixed;
  top: 90px;
  right: 24px;
  z-index: 2100;

  width: min(520px, calc(100vw - 48px));
  max-height: calc(100vh - 114px);

  pointer-events: none;
}

.route-direction-dialog {
  width: 100%;
  max-height: calc(100vh - 114px);
  overflow-y: auto;

  padding: 18px;
  border: 1px solid rgba(0, 229, 255, 0.55);
  border-radius: 12px;

  background: #17232c;
  color: #f7fbff;

  box-shadow: 0 22px 60px rgba(0, 0, 0, 0.55);

  pointer-events: auto;
}

.route-direction-dialog__header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 16px;
}

.route-direction-dialog__header h2 {
  margin: 3px 0 0;
  color: #8fd3ff;
  font-size: 1.08rem;
}

.route-direction-dialog__route {
  display: grid;
  gap: 5px;
  padding: 12px;
  border-radius: 7px;
  background: rgba(0, 229, 255, 0.1);
}

.route-direction-dialog__route span {
  color: #9fb0ba;
  font-size: 0.72rem;
  font-weight: 800;
  text-transform: uppercase;
}

.route-direction-dialog__route strong {
  color: #f7fbff;
  font-size: 0.9rem;
  overflow-wrap: anywhere;
}

.route-direction-dialog__help {
  margin: 13px 0;
  color: #dbe9f1;
  font-size: 0.76rem;
  line-height: 1.5;
}

.route-direction-dialog__definition-grid {
  display: grid;
  grid-template-columns: repeat(
    2,
    minmax(0, 1fr)
  );
  gap: 10px;
}

.route-direction-dialog__definition {
  display: grid;
  gap: 9px;
  padding: 13px;
  border: 1px solid
    rgba(143, 211, 255, 0.22);
  border-radius: 8px;
  background: rgba(7, 19, 26, 0.68);
}

.route-direction-dialog__step {
  display: grid;
  place-items: center;
  width: 25px;
  height: 25px;
  border-radius: 50%;
  background: #8fd3ff;
  color: #07131a;
  font-size: 0.76rem;
  font-weight: 900;
}

.route-direction-dialog__definition strong {
  color: #f7fbff;
  font-size: 0.82rem;
}

.route-direction-dialog__value {
  min-height: 34px;
  padding: 8px 9px;
  border-radius: 5px;
  background: rgba(255, 255, 255, 0.07);
  color: #9fb0ba;
  font-size: 0.74rem;
  font-weight: 800;
}

.route-direction-dialog__value--defined {
  background: rgba(102, 187, 106, 0.16);
  color: #b9f6ca;
}

.route-direction-dialog__definition button {
  width: 100%;
  min-height: 38px;
  padding: 8px 10px;
  white-space: normal;
}

.route-direction-dialog__actions {
  display: grid;
  grid-template-columns:
    minmax(0, 0.7fr)
    minmax(0, 1.3fr);
  gap: 9px;
  margin-top: 16px;
}

.route-direction-dialog__actions button {
  min-height: 42px;
  padding: 8px 12px;
  white-space: normal;
}

.route-direction-dialog__node-list {
  display: grid;
  gap: 6px;
}

.route-direction-dialog__node {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;

  min-height: 36px;
  padding: 6px 7px 6px 10px;

  border: 1px solid
    rgba(102, 187, 106, 0.28);
  border-radius: 6px;

  background: rgba(102, 187, 106, 0.13);
  color: #b9f6ca;

  font-size: 0.74rem;
  font-weight: 800;
}

.route-direction-dialog__node button {
  width: 28px;
  min-width: 28px;
  min-height: 28px;
  padding: 0;

  border-radius: 5px;

  color: #ffffff;
  background: rgba(255, 82, 82, 0.75);

  font-size: 1rem;
  line-height: 1;
}

.manual-reset-section {
  display: grid;
  gap: 9px;

  margin-top: 0;
  padding: 12px;

  border: 1px solid rgba(255, 82, 82, 0.38);
  border-radius: 8px;

  background: rgba(255, 82, 82, 0.08);
}

.manual-reset-section__description {
  margin: 0;

  color: #d6e0e6;
  font-size: 0.73rem;
  line-height: 1.45;
}

.manual-reset-button {
  width: 100%;
  min-height: 42px;
  padding: 9px 12px;

  border: 1px solid rgba(255, 82, 82, 0.75);
  border-radius: 6px;

  background: rgba(176, 38, 38, 0.86);
  color: #ffffff;

  font-size: 0.76rem;
  font-weight: 900;

  cursor: pointer;
}

.manual-reset-button:hover {
  background: rgba(211, 47, 47, 0.95);
  border-color: #ff8a80;
}

.manual-reset-button:focus-visible {
  outline: 2px solid #ff8a80;
  outline-offset: 2px;
}

.flow-preparation-status {
  display: grid;
  gap: 7px;
  margin-top: 10px;
  padding: 10px;

  border: 1px solid
    rgba(143, 211, 255, 0.25);
  border-radius: 7px;

  background: rgba(7, 19, 26, 0.58);
}

.flow-preparation-status__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;

  color: #dbe9f1;
  font-size: 0.73rem;
  font-weight: 800;
}

.flow-preparation-status__header strong {
  color: #8fd3ff;
  font-size: 0.76rem;
}

.flow-preparation-progress {
  width: 100%;
  height: 9px;
  overflow: hidden;

  border-radius: 999px;

  background: rgba(
    255,
    255,
    255,
    0.11
  );
}

.flow-preparation-progress__fill {
  width: 0;
  height: 100%;

  border-radius: inherit;

  background: #ffb300;

  transition:
    width 180ms ease,
    background-color 180ms ease;
}

.flow-preparation-progress__fill--ready {
  background: #66bb6a;
}

.flow-preparation-progress__fill--error {
  background: #ff5252;
}

@media (max-width: 820px) {
  .control-panels {
    top: auto;
    right: 12px;
    bottom: 12px;
    width: calc(100vw - 24px);
    max-height: calc(100vh - 24px);
  }

.route-direction-dialog__definition-grid {
  grid-template-columns: 1fr;
}

.route-direction-dialog__actions {
  grid-template-columns: 1fr;
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

.valve-management-panel {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  margin-top: 0.75rem;
}

.valve-management-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  max-height: 20rem;
  padding: 0.5rem;
  overflow-y: auto;
  border: 1px solid rgba(143, 211, 255, 0.25);
  border-radius: 0.5rem;
  background: rgba(0, 0, 0, 0.16);
}

.valve-management-item {
  display: grid;
  grid-template-columns: auto minmax(0, 1fr);
  align-items: start;
  gap: 0.65rem;
  padding: 0.65rem;
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 0.4rem;
  background: rgba(255, 255, 255, 0.04);
  cursor: pointer;
}

.valve-management-item:hover {
  border-color: rgba(143, 211, 255, 0.55);
  background: rgba(143, 211, 255, 0.08);
}

.valve-management-item:has(input:checked) {
  border-color: #8fd3ff;
  background: rgba(143, 211, 255, 0.14);
}

.valve-management-item input {
  width: 1rem;
  height: 1rem;
  margin: 0.15rem 0 0;
  accent-color: #8fd3ff;
  cursor: pointer;
}

.valve-management-item span {
  min-width: 0;
  font-size: 0.85rem;
  line-height: 1.35;
  overflow-wrap: anywhere;
  color: #e7f1f6;
}

.valve-management-list::-webkit-scrollbar {
  width: 0.45rem;
}

.valve-management-list::-webkit-scrollbar-track {
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.05);
}

.valve-management-list::-webkit-scrollbar-thumb {
  border-radius: 999px;
  background: rgba(143, 211, 255, 0.55);
}

.selected-tube-route-banner {
  position: fixed;
  top: 18px;
  left: 50%;
  z-index: 1200;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
  width: max-content;
  max-width: min(
    42rem,
    calc(100vw - 4rem)
  );
  padding: 0.7rem 1rem;
  border: 1px solid
    rgba(143, 211, 255, 0.7);
  border-radius: 0.6rem;
  background:
    rgba(13, 22, 28, 0.94);
  box-shadow:
    0 0.5rem 1.5rem
    rgba(0, 0, 0, 0.35);
  color: #ffffff;
  text-align: center;
  transform: translateX(-50%);
  pointer-events: none;
}

.selected-tube-route-banner__label {
  color: #8fd3ff;
  font-size: 0.72rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.selected-tube-route-banner strong {
  font-size: 0.95rem;
  line-height: 1.35;
  overflow-wrap: anywhere;
}

.automatic-valve-preview-backdrop {
  position: fixed;
  inset: 0;
  z-index: 5000;
  display: grid;
  place-items: center;
  padding: 1.5rem;
  background: rgba(0, 0, 0, 0.62);
  backdrop-filter: blur(0.35rem);
}

.automatic-valve-preview-dialog {
  display: flex;
  flex-direction: column;
  gap: 0.9rem;
  width: min(58rem, 100%);
  max-height: calc(100vh - 3rem);
  padding: 1.2rem;
  overflow: hidden;
  border: 1px solid rgba(143, 211, 255, 0.45);
  border-radius: 0.8rem;
  background: rgba(13, 22, 28, 0.98);
  box-shadow: 0 1rem 3rem rgba(0, 0, 0, 0.5);
  color: #ffffff;
}

.automatic-valve-preview-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
}

.automatic-valve-preview-header h2 {
  margin: 0.2rem 0 0;
}

.automatic-valve-preview-list {
  display: flex;
  flex-direction: column;
  gap: 0.7rem;
  min-height: 0;
  padding-right: 0.3rem;
  overflow-y: auto;
}

.automatic-valve-preview-item {
  padding: 0.85rem;
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 0.6rem;
  background: rgba(255, 255, 255, 0.04);
}

.automatic-valve-preview-item--safe {
  border-color: rgba(98, 214, 138, 0.42);
}

.automatic-valve-preview-item--warning {
  border-color: rgba(255, 183, 77, 0.5);
}

.automatic-valve-preview-item--accepted {
  border-color: #8fd3ff;
  background: rgba(143, 211, 255, 0.12);
}

.automatic-valve-preview-selection {
  display: flex;
  align-items: center;
  gap: 0.65rem;
  cursor: pointer;
}

.automatic-valve-preview-selection input {
  width: 1rem;
  height: 1rem;
  accent-color: #8fd3ff;
}

.automatic-valve-preview-details {
  display: grid;
  grid-template-columns:
    repeat(3, minmax(0, 1fr));
  gap: 0.65rem;
  margin: 0.8rem 0 0;
}

.automatic-valve-preview-details div {
  min-width: 0;
  padding: 0.55rem;
  border-radius: 0.4rem;
  background: rgba(0, 0, 0, 0.2);
}

.automatic-valve-preview-details dt {
  margin-bottom: 0.2rem;
  color: #8fd3ff;
  font-size: 0.72rem;
  font-weight: 700;
  text-transform: uppercase;
}

.automatic-valve-preview-details dd {
  margin: 0;
  overflow-wrap: anywhere;
  color: #f7fbff;
  font-size: 0.84rem;
}

@media (max-width: 700px) {
  .automatic-valve-preview-details {
    grid-template-columns:
      minmax(0, 1fr);
  }
}

.route-partial-block-icon {
  display: inline-grid;
  place-items: center;
  width: 1rem;
  height: 1rem;
  margin-right: 0.3rem;
  border: 1px solid
    rgba(255, 112, 67, 0.9);
  border-radius: 50%;
  background:
    rgba(255, 87, 34, 0.18);
  color: #ff7043;
  font-size: 0.68rem;
  line-height: 1;
  vertical-align: middle;
  cursor: help;
  box-shadow:
    0 0 0.4rem
    rgba(255, 87, 34, 0.28);
}

.saved-route-actions {
  display: grid;
  grid-template-columns:
    repeat(3, minmax(0, 1fr));

  grid-template-areas:
    "highlight visibility simulation"
    "direction sync reverse"
    "rename color protect"
    "arrows reset-color undo-merge"
    "delete delete delete";

  gap: 0.45rem;
  width: 100%;
  margin-top: 0.7rem;
}

.saved-route-actions button {
  width: 100%;
  min-width: 0;
  min-height: 2rem;
  padding: 0.35rem 0.45rem;
  overflow: hidden;
  font-size: 0.7rem;
  line-height: 1.15;
  text-overflow: ellipsis;
  white-space: normal;
}

.saved-route-action--highlight {
  grid-area: highlight;
}

.saved-route-action--visibility {
  grid-area: visibility;
}

.saved-route-action--reverse {
  grid-area: reverse;
}

.saved-route-action--direction {
  grid-area: direction;
}

.saved-route-action--sync {
  grid-area: sync;
}

.saved-route-action--rename {
  grid-area: rename;
}

.saved-route-action--color {
  grid-area: color;
}

.saved-route-action--reset-color {
  grid-area: reset-color;
}

.saved-route-action--protect {
  grid-area: protect;
}

.saved-route-action--simulation {
  grid-area: simulation;
}

.saved-route-action--undo-merge {
  grid-area: undo-merge;
}

.saved-route-action--delete {
  grid-area: delete;
}

.saved-route-action--arrows {
  grid-area: arrows;
}

@media (max-width: 700px) {
  .saved-route-actions {
    grid-template-columns:
      repeat(2, minmax(0, 1fr));

    grid-template-areas:
      "highlight visibility"
      "arrows reverse"
      "direction sync"
      "rename color"
      "reset-color protect"
      "simulation undo-merge"
      "delete delete";
  }
}

.saved-route-top-line {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.6rem;
  width: 100%;
  min-height: 1.15rem;
}

.saved-route-simulation-status {
  display: block;
  width: 0.72rem;
  height: 0.72rem;
  flex: 0 0 0.72rem;
  border: 2px solid
    rgba(255, 255, 255, 0.85);
  border-radius: 50%;
  background: #39d353;
  box-shadow:
    0 0 0.25rem
      rgba(57, 211, 83, 0.95),
    0 0 0.65rem
      rgba(57, 211, 83, 0.7);
}

.saved-route-actions--locked {
  grid-template-columns:
    repeat(2, minmax(0, 1fr));

  grid-template-areas:
    "highlight visibility"
    "protect simulation";
}

.saved-route-actions--locked button {
  min-height: 2.2rem;
}

.automatic-valve-calculation-status {
  display: grid;
  gap: 0.55rem;
  margin-top: 0.65rem;
  padding: 0.75rem;
  border: 1px solid
    rgba(143, 211, 255, 0.38);
  border-radius: 0.55rem;
  background:
    rgba(143, 211, 255, 0.09);
}

.automatic-valve-calculation-status__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  color: #dbe9f1;
  font-size: 0.76rem;
  font-weight: 800;
}

.automatic-valve-calculation-status__header strong {
  color: #8fd3ff;
}

.automatic-valve-calculation-progress {
  height: 0.55rem;
  overflow: hidden;
  border-radius: 999px;
  background:
    rgba(255, 255, 255, 0.12);
}

.automatic-valve-calculation-progress__fill {
  width: 0;
  height: 100%;
  border-radius: inherit;
  background: #8fd3ff;
  transition: width 0.2s ease;
}

.valve-designation-dropdown {
  position: relative;
  display: grid;
  gap: 0.4rem;
  width: 100%;
}

.valve-designation-dropdown__label {
  color: #b8c9d3;
  font-size: 0.75rem;
  font-weight: 800;
}

.valve-designation-dropdown__button {
  display: grid;
  grid-template-columns:
    minmax(0, 1fr) auto;
  align-items: center;
  gap: 0.6rem;
  width: 100%;
  min-height: 2.25rem;
  padding: 0.5rem 0.65rem;
  border: 1px solid
    rgba(255, 255, 255, 0.18);
  border-radius: 0.45rem;
  background: #f7fbff;
  color: #17242c;
  text-align: left;
}

.valve-designation-dropdown__button span:first-child {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.valve-designation-dropdown__arrow {
  color: #426579;
  font-size: 0.7rem;
}

.valve-designation-dropdown__menu {
  position: absolute;
  top: calc(100% + 0.35rem);
  right: 0;
  left: 0;
  z-index: 1200;
  display: grid;
  max-height: 16rem;
  overflow-y: auto;
  padding: 0.3rem;
  border: 1px solid
    rgba(143, 211, 255, 0.45);
  border-radius: 0.5rem;
  background: #17242c;
  box-shadow:
    0 0.75rem 1.8rem
    rgba(0, 0, 0, 0.45);
}

.valve-designation-dropdown__option {
  width: 100%;
  padding: 0.55rem 0.65rem;
  border: 0;
  border-radius: 0.35rem;
  background: transparent;
  color: #dbe9f1;
  text-align: left;
  font-size: 0.74rem;
  line-height: 1.3;
  white-space: normal;
  overflow-wrap: anywhere;
}

.valve-designation-dropdown__option:hover {
  background:
    rgba(143, 211, 255, 0.13);
}

.valve-designation-dropdown__option--selected {
  background:
    rgba(143, 211, 255, 0.22);
  color: #8fd3ff;
}

.selected-valve-summary {
  display: grid;
  gap: 0.35rem;
  margin-top: 0.55rem;
  padding: 0.65rem;
  border: 1px solid
    rgba(255, 255, 255, 0.12);
  border-radius: 0.45rem;
  background:
    rgba(255, 255, 255, 0.05);
}

.selected-valve-summary p {
  margin: 0;
  color: #dbe9f1;
  font-size: 0.75rem;
  line-height: 1.35;
}

.selected-valve-summary strong {
  color: #8fd3ff;
}

.route-creation-method-switch {
  display: grid;
  grid-template-columns:
    repeat(2, minmax(0, 1fr));
  gap: 0.25rem;
  margin-top: 0.65rem;
  padding: 0.25rem;
  border: 1px solid
    rgba(143, 211, 255, 0.28);
  border-radius: 0.55rem;
  background:
    rgba(255, 255, 255, 0.06);
}

.route-creation-method-switch button {
  min-width: 0;
  min-height: 2.4rem;
  padding: 0.5rem 0.6rem;
  border: 0;
  border-radius: 0.4rem;
  background: transparent;
  color: #b8c9d3;
  font-size: 0.74rem;
  font-weight: 900;
  line-height: 1.2;
}

.route-creation-method-switch
.route-creation-method-switch__option--active {
  background: #8fd3ff;
  color: #07131a;
  box-shadow:
    0 0 0 1px
      rgba(143, 211, 255, 0.4),
    0 0.3rem 0.8rem
      rgba(0, 0, 0, 0.24);
}

.route-creation-status {
  display: grid;
  gap: 0.3rem;
  margin-top: 0.65rem;
  padding: 0.65rem;
  border: 1px solid
    rgba(255, 255, 255, 0.12);
  border-radius: 0.45rem;
  background:
    rgba(255, 255, 255, 0.05);
}

.route-creation-status p {
  margin: 0;
  color: #dbe9f1;
  font-size: 0.75rem;
  line-height: 1.35;
}

.route-creation-status strong {
  color: #8fd3ff;
}

.route-creation-three-actions {
  display: grid;
  grid-template-columns:
    repeat(3, minmax(0, 1fr));
  gap: 0.45rem;
  width: 100%;
  margin-top: 0.65rem;
}

.route-creation-three-actions button {
  width: 100%;
  min-width: 0;
  min-height: 2.65rem;
  padding: 0.45rem 0.35rem;
  overflow-wrap: anywhere;
  font-size: 0.72rem;
  font-weight: 800;
  line-height: 1.15;
}

@media (max-width: 500px) {
  .route-creation-three-actions {
    grid-template-columns:
      repeat(3, minmax(0, 1fr));
  }

  .route-creation-three-actions button {
    padding-right: 0.25rem;
    padding-left: 0.25rem;
    font-size: 0.66rem;
  }
}

.route-edit-help {
  display: grid;
  gap: 0.55rem;
  margin-top: 0.65rem;
  padding: 0.75rem;
  border: 1px solid
    rgba(143, 211, 255, 0.3);
  border-radius: 0.55rem;
  background:
    rgba(143, 211, 255, 0.07);
}

.route-edit-help__title {
  margin: 0;
  color: #8fd3ff;
  font-size: 0.78rem;
  font-weight: 900;
}

.route-edit-help__steps {
  display: grid;
  gap: 0.35rem;
  margin: 0;
  padding-left: 1.25rem;
  color: #dbe9f1;
  font-size: 0.76rem;
  line-height: 1.4;
}

.route-edit-help__warning {
  margin: 0;
  padding: 0.55rem 0.65rem;
  border-left: 3px solid #ffb300;
  border-radius: 0.3rem;
  background:
    rgba(255, 179, 0, 0.11);
  color: #ffe0a3;
  font-size: 0.74rem;
  font-weight: 700;
  line-height: 1.4;
}

.selected-pipe-arrow-actions {
  display: grid;
  grid-template-columns:
    repeat(2, minmax(0, 1fr));
  gap: 0.45rem;
  width: 100%;
  margin-top: 0.55rem;
}

.selected-pipe-arrow-actions button {
  width: 100%;
  min-width: 0;
  min-height: 36px;
  padding: 0.45rem 0.55rem;
  border: 0;
  border-radius: 6px;
  background: #f7fbff;
  color: #111820;
  cursor: pointer;
  font: inherit;
  font-weight: 700;
  line-height: normal;
  white-space: normal;
  overflow-wrap: break-word;
}

.selected-pipe-arrow-actions button:hover:not(:disabled) {
  background: #d9f0ff;
}

.selected-pipe-arrow-actions button:disabled {
  background: #6f7376;
  color: #4c4f51;
  cursor: not-allowed;
  opacity: 0.75;
}

.configuration-reset-option {
  display: grid;
  gap: 0.55rem;
  padding: 0.75rem;
  border: 1px solid
    rgba(143, 211, 255, 0.25);
  border-radius: 0.55rem;
  background:
    rgba(143, 211, 255, 0.06);
}

.configuration-reset-option strong {
  color: #8fd3ff;
  font-size: 0.8rem;
  font-weight: 900;
}

.configuration-reset-option p {
  margin: 0;
  color: #d6e0e6;
  font-size: 0.73rem;
  line-height: 1.45;
}

.configuration-reset-option--danger {
  border-color:
    rgba(255, 82, 82, 0.42);
  background:
    rgba(255, 82, 82, 0.08);
}

.configuration-reset-option--danger strong {
  color: #ff9b8f;
}

.configuration-reset-button {
  width: 100%;
  min-height: 2.65rem;
  padding: 0.55rem 0.75rem;
  border-radius: 0.4rem;
  font-size: 0.76rem;
  font-weight: 900;
  cursor: pointer;
}

.configuration-reset-button--secondary {
  border: 1px solid
    rgba(143, 211, 255, 0.55);
  background:
    rgba(143, 211, 255, 0.16);
  color: #dff4ff;
}

.configuration-reset-button--secondary:hover {
  background:
    rgba(143, 211, 255, 0.25);
}

.configuration-reset-button--danger {
  border: 1px solid
    rgba(255, 82, 82, 0.75);
  background:
    rgba(176, 38, 38, 0.86);
  color: #ffffff;
}

.configuration-reset-button--danger:hover {
  background:
    rgba(211, 47, 47, 0.95);
}

.manual-valve-pipe-definition {
  display: grid;
  gap: 0.65rem;
}

.manual-valve-pipe-definition__summary {
  display: grid;
  gap: 0.35rem;
  padding: 0.7rem;
  border: 1px solid
    rgba(143, 211, 255, 0.28);
  border-radius: 0.5rem;
  background:
    rgba(143, 211, 255, 0.07);
}

.manual-valve-pipe-definition__summary p {
  margin: 0;
  color: #dbe9f1;
  font-size: 0.75rem;
  line-height: 1.4;
}

.manual-valve-pipe-definition__summary strong {
  color: #8fd3ff;
}

.manual-valve-pipe-definition__active {
  display: grid;
  gap: 0.55rem;
}

.manual-valve-pipe-definition__saved-actions {
  display: grid;
  grid-template-columns:
    repeat(3, minmax(0, 1fr));
  gap: 0.45rem;
}

.manual-valve-pipe-definition__saved-actions button {
  width: 100%;
  min-width: 0;
  min-height: 2.65rem;
  padding: 0.45rem;
  border: 0;
  border-radius: 6px;
  background: #f7fbff;
  color: #111820;
  font: inherit;
  font-weight: 700;
  line-height: normal;
  white-space: normal;
  overflow-wrap: break-word;
}

.manual-valve-pipe-definition__saved-actions button:disabled {
  background: #6f7376;
  color: #4c4f51;
  cursor: not-allowed;
  opacity: 0.75;
}

.manual-valve-pipe-definition__saved-actions
.flow-button--danger:not(:disabled) {
  background: #ffe3e3;
  color: #7a1010;
}

.selected-valve-control-actions {
  display: grid;
  grid-template-columns:
    repeat(2, minmax(0, 1fr));
  gap: 0.45rem;
  width: 100%;
  margin-top: 0.55rem;
}

.selected-valve-control-actions button {
  width: 100%;
  min-width: 0;
  min-height: 2.65rem;
  padding: 0.5rem 0.55rem;
  border: 0;
  border-radius: 6px;
  background: #f7fbff;
  color: #111820;
  cursor: pointer;
  font: inherit;
  font-weight: 700;
  line-height: normal;
}

.selected-valve-control-actions
button:hover:not(:disabled) {
  background: #d9f0ff;
}

.selected-valve-control-actions
button:disabled {
  background: #6f7376;
  color: #4c4f51;
  cursor: not-allowed;
  opacity: 0.75;
}

.route-direction-dialog__warning {
  margin: 0.75rem 0;
  padding: 0.7rem 0.8rem;
  border: 1px solid
    rgba(255, 179, 0, 0.55);
  border-left: 4px solid #ffb300;
  border-radius: 0.45rem;
  background:
    rgba(255, 179, 0, 0.12);
  color: #ffe0a3;
  font-size: 0.76rem;
  font-weight: 700;
  line-height: 1.45;
}
</style>