<script lang="js" frontend>
import Datepicker from 'vuejs-datepicker';

import Debug from '@doop/debug';

const $debug = Debug('@doop/search:searchInputTagDate').enable(true);

/**
* TODO: Docs
*/
app.component('searchInputTagDate', {
	components: {
		'date-picker': Datepicker
	},
	data() { return {
		rawValue: undefined,
	}},
	props: {
		value: {type: String},
		dateFormat: {type: String, default: 'DD/MM/YYYY'},
		disabled: {type: Boolean, default: false},
		placeholder: {type: String},
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
		<date-picker
			:bootstrap-styling="false"
			wrapper-class="form-control"
			:clear-button="true"
			:disabled="disabled"
			:placeholder="placeholder"
			:value="this.rawValue"
			@selected="handleChange"
		/>
	</div>
</template>
