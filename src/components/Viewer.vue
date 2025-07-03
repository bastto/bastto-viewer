<template>
    <!-- Loading Overlay -->
    <div v-if="isLoading" class="loading-overlay">
    <div class="loading-box">
        <div class="spinner"></div>
        <p class="loading-text">Loading {{ loadingFileName }}</p>
    </div>
    </div>
    <!-- Modal -->
    <div v-if="showModal" class="modal-backdrop">
    <div class="modal-content">
        <h3>Name your fragment file:</h3>
        <input v-model="fragmentName" placeholder="model.frag" />
        <div class="modal-actions">
        <button @click="showModal = false">Cancel</button>
        <button @click="downloadFragment">Download</button>
        </div>
    </div>
    </div>
    <div ref="containerRef" class="full-screen">
        <FloatingMenu
        @loadIfc="handleLoadIfc"
        @loadFrag="handleLoadFrag"
        @clearScene="handleClear"
        @downloadFrag="handleDownloadFrag"/>

    <!-- Hiden file inputs-->
        <input ref="ifcInput" type="file" accept=".ifc" style="display: none" @change="onIfcFileChange"/>
        <input ref="fragInput" type="file" accept=".frag" style="display: none" @change="onFragFileChange"/>
    </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import * as OBC from "@thatopen/components";
import * as FRAGS from "@thatopen/fragments";
import FloatingMenu from './FloatingMenu.vue'

const containerRef = ref<HTMLDivElement|null>(null);
const ifcInput = ref<HTMLInputElement | null>(null);
const fragInput = ref<HTMLInputElement | null>(null);
const showModal = ref(false);
const fragmentName = ref("model");
const isLoading = ref(false);
const loadingFileName = ref('');

let world: any;
let fragments: any;
let serializer: any;
let fragmentBytes: ArrayBuffer | null = null;
let model: FRAGS.FragmentsModel | undefined = undefined;
//export let selector : IfcSelector | null = null

onMounted(async () => {
    if (!containerRef.value) return;

    const components = new OBC.Components();
    const worlds = components.get(OBC.Worlds);
    world = worlds.create<
    OBC.SimpleScene,
    OBC.SimpleCamera,
    OBC.SimpleRenderer>();

    world.scene = new OBC.SimpleScene(components);
    world.scene.three.background = null;
    
    world.renderer = new OBC.SimpleRenderer(components, containerRef.value);
    world.camera = new OBC.OrthoPerspectiveCamera(components);
    
    components.init();
    world.camera.controls.setLookAt(-60, 30, -20, 0, 0, -20);

    world.scene.setup();

    const grids = components.get(OBC.Grids);
    grids.create(world);

    serializer = new FRAGS.IfcImporter();
    serializer.wasm = {
        absolute: true,
        path: "https://unpkg.com/web-ifc@0.0.68/",
    };

    const workerUrl = "https://thatopen.github.io/engine_fragment/resources/worker.mjs";
    const fetchedWorker = await fetch(workerUrl);
    const workerText = await fetchedWorker.text();
    const workerFile = new File([new Blob([workerText])], "worker.mjs", {
        type: "text/javascript",
    });
    
    const workerBlobUrl = URL.createObjectURL(workerFile);
    fragments = new FRAGS.FragmentsModels(workerBlobUrl);
});

const handleLoadIfc = () => {
    ifcInput.value?.click();
};

const handleLoadFrag = () => {
    fragInput.value?.click();
};

const handleClear = async() => {
    const ids = [...fragments.models.list.values()].map((m: any) => m.modelId);
    for (const id of ids) {
        await fragments.disposeModel(id);
    }
};

const onIfcFileChange = async(e:Event)=>{
    const file = (e.target as HTMLInputElement).files?.[0];
    if (!file) return;

    isLoading.value = true;
    loadingFileName.value = file.name;

    try{
        const buffer = await file.arrayBuffer();
        const ifcBytes = new Uint8Array(buffer);
        fragmentBytes = await serializer.process({bytes:ifcBytes});
        world.camera.controls.addEventListener("rest", () => fragments?.update(true));
        world.camera.controls.addEventListener("update", () => fragments?.update());
        model = await fragments.load(fragmentBytes, {modelId: file.name});
        model?.useCamera(world.camera.three);
        world.scene.three.add(model?.object);
        await fragments.update(true);
    } catch (err) {
        console.error("Failed to load IFC:", err);
        alert("Error loading file");
    }finally{
        isLoading.value = false;
    }
};

const onFragFileChange = async (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;

    isLoading.value = true;
    loadingFileName.value = file.name;

    try{
    
        world.camera.controls.addEventListener("rest", () => fragments?.update(true));
        world.camera.controls.addEventListener("update", () => fragments?.update());
    
        const buffer = await file.arrayBuffer();
        const model = await fragments.load(buffer, { modelId: "example" });
        model.useCamera(world.camera.three);
        world.scene.three.add(model.object);
        await fragments.update(true);
        const tree = await model.getSpatialStructure()
        console.log(tree)
    } catch (err) {
        console.error("Failed to load IFC:", err);
        alert("Error loading file");
    }finally{
        isLoading.value = false;
    }
};

const handleDownloadFrag = () => {
  if (!fragmentBytes) {
    alert("No fragment loaded yet.");
    return;
  }
  showModal.value = true;
};

const downloadFragment = () => {
  if (!fragmentBytes) 
  {
    console.error("No fragments to download.");
    return;
  }
  const blob = new Blob([fragmentBytes], { type: "application/octet-stream" });
  const file = new File([blob], `${fragmentName.value || 'model'}.frag`);
  const a = document.createElement('a');
  a.href = URL.createObjectURL(file);
  a.download = file.name;
  a.click();
  URL.revokeObjectURL(a.href);
  showModal.value = false;
};
</script>

<style scoped>
.modal-backdrop {
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

.modal-content {
    background: white;
    padding: 20px;
    border-radius: 10px;
    width: 280px;
    text-align: center;
}

.modal-content input {
    width: 100%;
    margin-top: 10px;
    padding: 6px;
    border: 1px solid #00a3e0;
    border-radius: 4px;
}

.modal-actions {
    display: flex;
    justify-content: space-between;
    margin-top: 15px;
}

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

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid rgba(255,255,255,0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 12px;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>