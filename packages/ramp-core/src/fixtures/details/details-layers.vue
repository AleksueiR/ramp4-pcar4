<template>
    <panel-screen>
        <template #header>
            {{ $t('details.title') }}
        </template>
        <template #controls>
            <close @click="panel.close()"></close>
        </template>
        <template #content>
            <div class="p-5">{{ $t('details.layers.found', { numResults: payloadResults, numLayers: payload.length }) }}</div>
            <div
                class="px-20 py-10 text-md flex hover:bg-gray-200 cursor-pointer"
                v-for="(item, idx) in payload"
                :key="idx"
                @click="openResult(idx)"
            >
                <div class="truncate">
                    {{ layerInfo(idx) }}
                </div>
                <div class="flex-auto"></div>
                <div class="px-5">{{ item.items.length }}</div>
            </div>
        </template>
    </panel-screen>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator';
import { Get, Sync, Call } from 'vuex-pathify';
import { DetailsStore } from './store';

import { PanelInstance } from '@/api';
import { IdentifyResult } from 'ramp-geoapi';
import BaseLayer from 'ramp-geoapi/dist/layer/BaseLayer';

@Component({})
export default class DetailsLayersV extends Vue {
    @Prop() panel!: PanelInstance;
    @Get(DetailsStore.payload) payload!: IdentifyResult[];
    @Get('layer/layers') layers!: BaseLayer[];
    @Get('layer/getLayerByUid') getLayerByUid!: (uid: string) => BaseLayer | undefined;

    /**
     * Switches the panel screen to display the data for a given result.
     */
    openResult(index: number) {
        if (this.getLayerByUid(this.payload[index].uid)!.layerType === 'ogcWms') {
            // skip results screen for wms layers
            this.panel.show({ screen: 'details-screen-item', props: { resultIndex: index, layerType: 'ogcWms' , itemIndex: 0} });
        } else {
            this.panel.show({ screen: 'details-screen-result', props: { resultIndex: index } });
        }
    }

    layerInfo(idx: number) {
        const layerInfo = this.payload[idx];

        // Check to see if there is a custom template defined for the selected layer.
        let item: BaseLayer | undefined = this.layers
            .map(layer => {
                let layerNode = layer.getLayerTree();

                if (!layerNode) return;

                // Determine if the selected UID is a child of this layer.
                if (layerNode.findChildByUid(layerInfo.uid) !== undefined) {
                    return layer;
                }
            })
            .filter(node => node !== undefined)[0];

        if (!item) return;

        return item.getName() ? item.getName() : item.id;
    }

    /**
     * Calculates the total number of results received by identify.
     */
    get payloadResults(): number {
        return this.payload.map(r => r.items.length).reduce((a, b) => a + b, 0);
    }
}
</script>

<style lang="scss"></style>
