<script lang="js" frontend>
import Debug from '@doop/debug';

const $debug = Debug('@doop/search').enable(true);

/**
* TODO: Docs
*/
app.component('searchInputTagDate', {
	data() { return {
		date: undefined,
	}},
	props: {
		value: {type: String},
		dateFormat: {type: String, default: 'DD/MM/YYYY'},
	},
	methods: {
		handleChange(e) {
			$debug('handleChange', e);
			this.date = e;
			this.encodeQuery();
		},


		/**
		* Compute local state into a search query (also set the search query display)
		*/
		encodeQuery() {
			$debug('encodeQuery', 'input', this.value);
			this.$emit('change', moment(this.date).format(this.dateFormat));
		},


		/**
		* Decode a string query into local settings
		* This function mutates `tagValues` to match the incomming queryString + `fuzzyQuery` with human specified parts
		* @param {string} query String query to decode back into its component parts
		*/
		decodeQuery(query) {
			$debug('decodeQuery', query);

			this.date = this.value ? moment(this.value, this.dateFormat).toDate() : undefined;
			$debug('date', this.date);
		},
	},

	created() {
		this.$debug().enable(true);
	},
});
</script>

<template>
	<div class="search-input-tag-date">
		<v-date
			:value="this.value"
			@selected="handleChange"
			:clear-button="true"
		/>
	</div>
</template>

<style lang="scss">
.search-input-tag-date {

}
</style>
