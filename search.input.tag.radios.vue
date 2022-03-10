<script lang="js" frontend>
import Debug from '@doop/debug';

const $debug = Debug('@doop/search:searchInputTagRadios').enable(true);

/**
* TODO: Docs
*/
app.component('searchInputTagRadios', {
	data() { return {
		rawValue: {},
	}},
	props: {
		value: {type: String},
		options: {type: Array},
	},
	methods: {
		setOption(id) {
			$debug('setOption', this.rawValue, id);
			this.rawValue = id;
			this.encodeQuery();
		},


		/**
		* Compute local state into a search query (also set the search query display)
		*/
		encodeQuery() {
			$debug('encodeQuery', this.rawValue);

			this.$emit('change', this.rawValue);
		},
	},

	created() {
		this.$debug().enable(true);

		this.$watch('value', () => {
			if (this.value) this.rawValue = this.value;
		}, { immediate: true });
	},
});
</script>

<template>
	<div class="search-input-tag-radios">
		<div v-for="option in options" :key="option.id" class="form-check mr-3">
			<input
				class="form-check-input"
				type="radio"
				:id="`${_uid}-${option.id}`"
				:name="_uid"
				:checked="rawValue == option.id"
				@change="setOption(option.id)"
			/>
			<label
				class="form-check-label"
				:for="`${_uid}-${option.id}`"
			>
				{{option.title}}
			</label>
		</div>
	</div>
</template>

<style lang="scss">
.search-input-tag-radios {

}
</style>
