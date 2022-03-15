<script lang="js" frontend>
import Debug from '@doop/debug';

const $debug = Debug('@doop/search:searchInputTagRangeMulti').enable(true);

import Multirange from 'multirange';
import 'multirange/multirange.css';

//import rangesliderJs from 'rangeslider-js';
//import './node_modules/rangeslider-js/dist/styles.min.css';

//import './libs/wrunner/wrunner-jquery.js';
//import './libs/wrunner/wrunner-native.js';
//import './libs/wrunner/themes/wrunner-default-theme.css';

//import './libs/range-slider/rSlider.min.js';
//import './libs/range-slider/rSlider.js';
//import './libs/range-slider/rSlider.min.css';

/**
* TODO: Docs
*/
app.component('searchInputTagRangeMulti', {
	data() { return {
		rawValue: undefined,
		//slider: undefined,
	}},
	props: {
		value: {type: String},
		min: {type: Number, default: 0},
		max: {type: Number, default: 100},
		// NOTE: Currently, no browser fully supports these features.
		// Firefox doesn't support hash marks and labels at all, for example,
		// while Chrome supports hash marks but doesn't support labels.
		// Version 66 (66.0.3359.181) of Chrome supports labels but the <datalist> tag has to be styled with CSS as its display property is set to none by default, hiding the labels.
		// @see https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range
		tickmarks: {type: Array},
	},
	methods: {
		handleChange(e) {
			$debug('handleChange', e);
			this.rawValue = e.target.value;
			this.encodeQuery();
		},


		/**
		* Compute local state into a search query (also set the search query display)
		*/
		encodeQuery() {
			$debug('encodeQuery', this.rawValue);
			this.$emit('change', this.rawValue.replace(',','-'));
		},
	},

	mounted() {
		Multirange.init();

		//rangesliderJs.create($(this.$el).find('input[type="range"]'));
		//$(this.$el).find('input[type="range"]').wRunner({
		//$(`#search-input-tag-range-multi-${this._uid}`).wRunner({

		/*
		//$(`#search-input-tag-range-multi-${this._uid}`).wRunner({
		this.slider = wRunner({
			roots: document.getElementById(`search-input-tag-range-multi-${this._uid}`),
			step: 1,
			type: 'range',
			limit: {minLimit: this.min, maxLimit: this.max},
			divisionsCount: 5,
			//rangeValue: { minValue: 20, maxValue: 80 }
		});
		//});
		*/
		/*
		this.slider = new rSlider({
			target: `#search-input-tag-range-multi-${this._uid}`,
			values: (_.isArray(this.tickmarks)) ? this.tickmarks : {min: this.min, max: this.max},
			range: true,
			tooltip: false,
			scale: true,
			labels: true,
			// TODO: Range, split on "-"
			//set: this.rawValue,
		});
		// TODO: Not correct size until after "resize"
		*/
	},

	created() {
		this.$debug().enable(true);

		this.$watch('value', () => {
			if (this.value) this.rawValue = this.value.replace('-',',');
		}, { immediate: true });
	},
	/*
	beforeDestroy() {
		this.slider.destroy();
	},
	*/
});
</script>

<template>
	<div class="search-input-tag-range-multi">
		<div class="form-control">
			<!--
			wRunner
			<input
				:id="`search-input-tag-range-multi-${_uid}`"
				type="range"
			/>
			-->
			<!--
			rSlider
			<input type="text" :id="`search-input-tag-range-multi-${_uid}`" />
			-->
			<!--
			Multirange
			// FIXME: Responding only to events on the original element
			-->
			<input
				:id="`search-input-tag-range-multi-${_uid}`"
				type="range"
				multiple
				:min="min"
				:max="max"
				:list="`tickmarks_${_uid}`"
				:value="this.rawValue"
				@change="handleChange"
			/>
			<datalist v-if="tickmarks" :id="`tickmarks_${_uid}`">
				<option
					v-for="(tick, index) in tickmarks" :key="index"
					:value="tick"
					:label="tick">
				</option>
			</datalist>
		</div>
	</div>
</template>

<style lang="scss">
.search-input-tag-range-multi {
	.form-control {
		display: inline-block;
	}

	datalist {
		display: flex;
		justify-content: space-between;
		padding-left: 0.15em;
		padding-right: 0.15em;
	}

	input {
		width: 100%;
	}

	/*
	input[type="range"].multirange.ghost {
		//--track-default: rgb(157, 150, 142);
		--track-default: transparent;
		--track-background: linear-gradient(to right,
				var(--track-default) var(--low), var(--range-color) 0,
				var(--range-color) var(--high), transparent 0
			) no-repeat 0 45% / 100% 40%;
	}
	*/

	// FIXME: Strip all styles from original and ghost, create another control behind them purely as a backdrop?

	/*
	input[type="range"].multirange.ghost::-webkit-slider-thumb {
		-webkit-appearance: none;
		background-color: #000;
	}
	*/

	/*
	input[type="range"].multirange.ghost::-webkit-slider-runnable-track {
		-webkit-appearance: none;
		height: 10px;
		color: #13bba4;
		margin-top: -1px;
	}
	*/

	/*
	input[type="range"].multirange.ghost,
	input[type="range"].multirange.ghost::-webkit-slider-runnable-track {
		-webkit-appearance: none;
	}

	input[type="range"].multirange.ghost::-webkit-slider-runnable-track {
		border: 0px none rgb(157, 150, 142);
		outline: rgb(157, 150, 142) none 0px;
		height: 22px;
	}
	*/

	/*
	input[type="range"].multirange.ghost::-moz-range-track {
		appearance: none;
	}
	*/

	/*
	input[type="range"].multirange.ghost::-webkit-slider-runnable-track {
		background: var(--track-background);
	}

	input[type="range"].multirange.ghost::-moz-range-track {
		background: var(--track-background);
	}
	*/

	// Fix width overflow when original is absolute positioned {{{
	input[type=range].multirange.original {
		position: relative;
	}

	input[type=range].multirange.ghost {
		position: relative;
		top: -24px;
	}
	// }}}
}
</style>
