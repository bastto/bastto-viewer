<template>
    <div ref="containerRef" class="full-screen">
        <bim-grid id="appGrid"></bim-grid>
    </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import * as OBC from "@thatopen/components";
import * as FRAGS from "@thatopen/fragments";
import * as BUI from "@thatopen/ui";
import * as BUIC from "@thatopen/ui-obc";

const containerRef = ref<HTMLDivElement|null>(null);

let world: any;
let serializer: any;

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
        path: "https://unpkg.com/web-ifc@0.0.68/",
    };

    const ifcLoader = components.get(OBC.IfcLoader);
    await ifcLoader.setup({autoSetWasm: true});

    const fragments = components.get(OBC.FragmentsManager);
    fragments.init(
        "/worker.mjs",
    );
    

    world.camera.controls.addEventListener("rest", () =>
        fragments.core.update(true),
    );

    fragments.list.onItemSet.add(async ({ value: model }) => {
        model.useCamera(world.camera.three);
        world.scene.three.add(model.object);
        await fragments.core.update(true);
    });

    const [modelsList] = BUIC.tables.modelsList({
        components,
        metaDataTags: ["schema"],
        actions: { download: true },
    });

    const panel = BUI.Component.create(() => {
        const [loadFragBtn] = BUIC.buttons.loadFrag({ components });
        const [loadIfcBtn] = BUIC.buttons.loadIfc({components});

        return BUI.html`
            <bim-panel label="IFC Models">
                <bim-panel-section label="Importing">
                    ${loadFragBtn}
                    ${loadIfcBtn}
                </bim-panel-section>
                <bim-panel-section icon="mage:box-3d-fill" label="Loaded Models">
                    ${modelsList}
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

    app.layout = "main";});
</script>

<style scoped>
</style>