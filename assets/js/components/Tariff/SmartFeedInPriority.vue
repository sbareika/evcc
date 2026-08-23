<template>
	<SmartTariffBase
		v-bind="labels"
		:current-limit="currentLimit"
		:last-limit="lastLimit"
		:currency="currency"
		:apply-all="multipleLoadpoints"
		:possible="possible"
		:tariff="tariff"
		:form-id="formId"
		:is-slot-active="isSlotActive"
		options-extra-high
		options-start-at-zero
		limit-direction="above"
		highlight-color="text-warning"
		@save-limit="saveLimit"
		@delete-limit="deleteLimit"
		@apply-to-all="applyToAll"
	/>
	<template v-if="possible">
		<p>{{ $t("smartFeedInPriority.dynamicDescription") }}</p>
		<div class="row mb-3 align-items-center" style="max-width: 1000px">
			<label :for="formId + 'Dynamic'" class="col-sm-4 col-form-label pt-0 pt-sm-2">
				{{ $t("smartFeedInPriority.dynamic") }}
			</label>
			<div class="col-sm-8 col-lg-4 pe-lg-0">
				<div class="form-check form-switch m-0">
					<input
						:id="formId + 'Dynamic'"
						class="form-check-input"
						type="checkbox"
						role="switch"
						:checked="dynamic"
						@change="toggleDynamic"
					/>
				</div>
			</div>
		</div>
	</template>
</template>

<script lang="ts">
import { defineComponent, type PropType } from "vue";
import SmartTariffBase from "./SmartTariffBase.vue";
import api from "@/api";
import { type CURRENCY } from "@/types/evcc";
import { setLoadpointLastSmartFeedInPriorityLimit } from "@/uiLoadpoints";

export default defineComponent({
	name: "SmartFeedInPriority",
	components: { SmartTariffBase },
	props: {
		currentLimit: {
			type: [Number, null] as PropType<number | null>,
			required: true,
		},
		lastLimit: Number,
		currency: String as PropType<CURRENCY>,
		loadpointId: String,
		multipleLoadpoints: Boolean,
		possible: Boolean,
		tariff: Array,
		dynamic: Boolean,
	},

	computed: {
		formId(): string {
			return `smartFeedInPriority-${this.loadpointId}`;
		},
		labels() {
			const t = (key: string) => this.$t(`smartFeedInPriority.${key}`);
			return {
				title: t("title"),
				description: t("description"),
				limitLabel: t("priceLimit"),
				activeHoursLabel: t("activeHoursLabel"),
				currentPriceLabel: t("priceLabel"),
				resetWarningKey: "smartFeedInPriority.resetWarning",
			};
		},
	},
	methods: {
		isSlotActive(value: number | undefined): boolean {
			if (value === undefined || this.currentLimit === null) {
				return false;
			}
			// Smart feed-in priority: pause when rates are above or equal to limit
			return value >= this.currentLimit;
		},
		async saveLimit(limit: number, active: boolean) {
			// save last selected value to be suggest again when reactivating limit
			this.saveLastLimit(limit);

			if (!active) return;

			const url = `loadpoints/${this.loadpointId}/smartfeedinprioritylimit`;
			await api.post(`${url}/${encodeURIComponent(limit)}`);
		},
		saveLastLimit(limit: number) {
			if (this.loadpointId) {
				setLoadpointLastSmartFeedInPriorityLimit(this.loadpointId, limit);
			}
		},
		async deleteLimit() {
			// save last selected value to be suggest again when reactivating limit
			this.saveLastLimit(this.currentLimit || 0);

			const url = `loadpoints/${this.loadpointId}/smartfeedinprioritylimit`;
			await api.delete(url);
		},
		async applyToAll(selectedLimit: number | null) {
			if (selectedLimit === null) {
				await api.delete("smartfeedinprioritylimit");
			} else {
				await api.post(`smartfeedinprioritylimit/${encodeURIComponent(selectedLimit)}`);
			}
		},
		async toggleDynamic(e: Event) {
			const value = (e.target as HTMLInputElement).checked;
			await api.post(`loadpoints/${this.loadpointId}/smartfeedinprioritydynamic/${value}`);
		},
	},
});
</script>
