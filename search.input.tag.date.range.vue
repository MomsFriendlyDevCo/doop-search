<script lang="js" frontend>
import Datepicker from 'vuejs-datepicker';

import Debug from '@doop/debug';

const $debug = Debug('@doop/search:searchInputTagDateRange').enable(true);

/**
* TODO: Docs
*/
app.component('searchInputTagDateRange', {
	components: {
		'date-picker': Datepicker
	},
	data() { return {
		rawValue: {
			start: undefined,
			finish: undefined,
		},
	}},
	props: {
		value: {type: String},
		dateFormat: {type: String, default: 'DD/MM/YYYY'},
		disabled: {type: Boolean, default: false},
		placeholder: {type: String},
		seperator: {type: String, default: '-'},
	},
	methods: {
		handleChange(path, e) {
			$debug('handleChange', path, e);
			this.$setPath(this.rawValue, path, e);
			this.encodeQuery();
		},


		/**
		* Compute local state into a search query (also set the search query display)
		*/
		encodeQuery() {
			$debug('encodeQuery', 'input', this.rawValue);

			// Special dataRange default output function
			const out = _.isEmpty(this.rawValue) ? null // Uninitalized dates
			: !this.rawValue.start && !this.rawValue.finish ? null // No dates
			: this.rawValue.start && this.rawValue.finish ? // Has both start + end
				//this.name + ':'
					moment(this.rawValue.start).format(this.dateFormat)
					+ this.seperator
					+ moment(this.rawValue.finish).format(this.dateFormat)
			//: this.rawValue.start && this.startOnlyTag ? // Only has start + alternate starting tag
			//	this.startOnlyTag + ':' + 
			//		moment(this.rawValue.start).format(this.dateFormat)
			: this.rawValue.start ? // Only has start
				//this.name + ':'
					moment(this.rawValue.start).format(this.dateFormat)
					+ this.seperator
			//: this.rawValue.finish && this.endOnlyTag ? // Only has end + alternate ending tag
			//	this.endOnlyTag + ':' + 
			//		moment(this.rawValue.finish).format(this.dateFormat)
			: this.rawValue.finish ? // Only has end
				//this.name + ':'
					this.seperator
					+ moment(this.rawValue.finish).format(this.dateFormat)
			: null; // All other cases
			$debug('encodeQuery', 'output', out);

			if (out) this.$emit('change', out);
		},
	},

	created() {
		this.$debug().enable(true);

		this.$watch('value', () => {
			if (!this.value) return;

			const values = this.value.split(this.seperator);
			this.rawValue.start = values[0];
			this.rawValue.finish = values[1];

			$debug('$watch', 'value', this.value, this.rawValue);
		}, { immediate: true });
	},
});
</script>

<template>
	<div class="search-input-tag-date-range row">
		<div class="col-5">
			<date-picker
				:bootstrap-styling="false"
				wrapper-class="form-control"
				:clear-button="true"
				:disabled="disabled"
				:placeholder="placeholder"
				:value="this.rawValue.start"
				@selected="handleChange('start', $event)"
			/>
		</div>
		<div class="col-2 text-center">to</div>
		<div class="col-5">
			<date-picker
				:bootstrap-styling="false"
				wrapper-class="form-control"
				:clear-button="true"
				:disabled="disabled"
				:placeholder="placeholder"
				:value="this.rawValue.finish"
				@selected="handleChange('finish', $event)"
			/>
		</div>
	</div>
</template>
