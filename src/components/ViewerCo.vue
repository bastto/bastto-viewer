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
        ></input>
    </div>

    <a href="https://github.com/bastto" target="_blank" class="corner-logo">
        <img src="/src/assets/bastto-logo.svg" alt="Logo" class="app-logo" />
        <img src="/src/assets/bastto-logo-name.svg" alt="Full Logo" class="logo-full" />
    </a>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import * as OBC from "@thatopen/components";
import * as FRAGS from "@thatopen/fragments";
import * as BUI from "@thatopen/ui";
import * as BUIC from "@thatopen/ui-obc";
import * as OBCF from "@thatopen/components-front"

const containerRef = ref<HTMLDivElement|null>(null);
const ifcInput = ref<HTMLInputElement | null>(null);
const isLoading = ref(false);
const loadingProgress = ref(0);
const loadingFileName = ref('');

let world: any;
let serializer: any;
let fragmentManager: OBC.FragmentsManager;
let fragmentBytes: ArrayBuffer | null = null;

onMounted(async () => {
    if (!containerRef.value) return;

    BUI.Manager.init();

    const components = new OBC.Components();
    const worlds = components.get(OBC.Worlds);
    world = worlds.create<
    OBC.SimpleScene,
    OBC.SimpleCamera,
    OBC.SimpleRenderer>();

    world.scene = new OBC.SimpleScene(components);
    world.scene.three.background = null;
    
    const viewport = document.createElement("bim-viewport");
    world.renderer = new OBC.SimpleRenderer(components, viewport);
    world.camera = new OBC.OrthoPerspectiveCamera(components);
    
    components.init();
    world.camera.controls.setLookAt(-60, 30, -20, 0, 0, -20);

    world.scene.setup();

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
    console.log(ifcLoader);

    fragmentManager = components.get(OBC.FragmentsManager);
    fragmentManager.init(
        "/worker.mjs",
    );
    
    world.camera.controls.addEventListener("rest", () =>
    fragmentManager.core.update(true),
    );

    
    
    fragmentManager.list.onItemSet.add(async ({ value: model }) => {
        model.useCamera(world.camera.three);
        world.scene.three.add(model.object);
        await fragmentManager.core.update(true);
    });
    
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
    
    highlighter.events.select.onHighlight.add((modelIdMap) => {
        updatePropertiesTable({ modelIdMap });
    });
    
    highlighter.events.select.onClear.add(() =>
        updatePropertiesTable({ modelIdMap: {} }),
    );

    propertiesTable.preserveStructureOnFilter = true;
    propertiesTable.indentationInText = false;

    const panel = BUI.Component.create(() => {
        const [loadFragBtn] = BUIC.buttons.loadFrag({ components });
    
        const onSearchSpatialTree = (e: Event) => {
            const input = e.target as BUI.TextInput;
            spatialTree.queryString = input.value;
        };

        const onTextInput = (e: Event)=> {
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
});

const openIfcDialog = () => ifcInput.value?.click();

const convertIFC = async (e:Event) => {
    const file = (e.target as HTMLInputElement).files?.[0];
    if (!file ) return;

    isLoading.value = true;
    loadingProgress.value = 0;
    loadingFileName.value = file.name;

    try
    {
        const buffer = await file.arrayBuffer();
        const ifcBytes = new Uint8Array(buffer);
    
        fragmentBytes = await serializer.process({
            bytes: ifcBytes,
            progressCallback: (progress) =>{
                if (loadingProgress) loadingProgress.value = Math.round(progress*100);
            },
        });
    
        if (!fragmentBytes) return;
        const model = await fragmentManager.core.load(fragmentBytes, {modelId: file.name});
    } catch(err)
    {
        console.error("IFC conversion failed:", err);
        alert("Failed to convert IFC file. Please try again.");
    } finally
    {
        isLoading.value = false;
    }


}

</script>

<style scoped>
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 42, 58, 0.6); /* #002a3a @ 60% */
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
  width: 48px;
  height: 48px;
  margin-bottom: 12px;
}

.loading-text {
  font-size: 1rem;
  font-weight: 500;
}

.corner-logo {
  position: fixed;
  bottom: 32px;
  left: 32px;
  border-radius: 5px;
  height: 64px;
  width: 64px;
  z-index: 1001;
  opacity: 1;
}

.corner-logo img{
    height: 100%;
    border-radius: 5px;
    transition: all 0.3s ease;
    position: absolute;
    width: auto;
}

.app-logo{
    opacity: 1;
}

.logo-full{
    opacity: 0;
}

.corner-logo:hover .app-logo{
    opacity: 0;
}

.corner-logo:hover .logo-full{
    opacity: 1;
}
</style>