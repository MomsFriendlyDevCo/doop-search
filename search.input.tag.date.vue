<script lang="js" frontend>
import Debug from '@doop/debug';

const $debug = Debug('@doop/search:searchInputTagDate').enable(true);

/**
* TODO: Docs
*/
app.component('searchInputTagDate', {
	data() { return {
		rawValue: undefined,
	}},
	props: {
		value: {type: String},
		dateFormat: {type: String, default: 'DD/MM/YYYY'},
	},
	methods: {
		handleChange(e) {
			$debug('handleChange', e);
			this.rawValue = e;
			this.encodeQuery();
		},


		/**
		* Compute local state into a search query (also set the search query display)
		*/
		encodeQuery() {
			$debug('encodeQuery', this.rawValue, this.dateFormat);
			this.$emit('change', moment(this.rawValue).format(this.dateFormat));
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
	<div class="search-input-tag-date">
		<v-date
			:value="this.rawValue"
			@selected="handleChange"
			:clear-button="true"
		/>
	</div>
</template>

<style lang="scss">
.search-input-tag-date {

}
</style>
