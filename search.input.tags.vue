<script lang="js" frontend>
import Debug from '@doop/debug';

const $debug = Debug('@doop/search').enable(true);

/**
* TODO: Docs
*/
app.component('searchInputTags', {
	data() { return {
		tagValues: {}, // Lookup values for each tag
	}},
	props: {
		values: {type: Object},
		tags: {type: Array, required: true},
	},
	methods: {
		/**
		* Set the value of a search tag
		* @param {string|array} Path Path relative to tagValues to set within
		* @param {*} value The value to set
		*/
		setTagValue(path, value) {
			$debug('setTagValue', path, value);
			this.$setPath(this.tagValues, path, value);
			this.encodeQuery();
		},


		/**
		* Compute local state into a search query (also set the search query display)
		*/
		encodeQuery() {
			$debug('encodeQuery', 'input', this.tags);

			// Extract values from hashmap and mutate tags definition
			this.$props.tags.forEach(tag => tag.value = this.tagValues[tag.tag]);
			$debug('encodeQuery', 'output', this.tags);
			this.$emit('change', this.tags);
		},
	},

	created() {
		$debug('created');

		// Initialise from provided values or default
		this.$props.tags.forEach(tag => {
			if (this.values[tag.tag]) {
				$debug('value', this.values[tag.tag]);
				this.$set(this.tagValues, tag.tag, this.values[tag.tag]);
			} else if (tag.default) {
				$debug('default', tag.default);
				this.$set(this.tagValues, tag.tag, tag.default);
			}
		});
	},
});
</script>

<template>
	<div class="search-input-tags">
		<div v-for="tag in tags.filter(ct => ct.type != 'hidden')" :key="tag.tag" class="form-group">
			<label class="col-sm-3 col-form-label">{{tag.title}}</label>
			<div class="col-9 mt-2">
				<!-- type='digest' {{{ -->
				<search-input-tag-digest v-if="tag.type == 'digest'"
					:value="tagValues[tag.tag]"
					:options="tag.digest"
					@change="setTagValue(tag.tag, $event)"
				/>
				<!-- }}} -->
				<!-- type='date' {{{ -->
				<search-input-tag-date v-else-if="tag.type == 'date'"
					:date-format="tag.dateFormat"
					:value="tagValues[tag.tag]"
					@change="setTagValue(tag.tag, $event)"
				/>
				<!--
				-->
				<!--v-date
					v-else-if="tag.type == 'date'"
					:value="tagValues[tag.tag]"
					@selected="setTagValue(tag.tag, $event)"
					:clear-button="true"
				/-->
				<!-- }}} -->
				<!-- type='dateRange' {{{ -->
				<div v-else-if="tag.type == 'dateRange'" class="row">
					<div class="col-5">
						<v-date
							:value="tagValues[tag.tag][0]"
							@selected="setTagValue([tag.tag, 0], $event)"
							:clear-button="true"
						/>
					</div>
					<div class="col-2 text-center">to</div>
					<div class="col-5">
						<v-date
							:value="tagValues[tag.tag][1]"
							@selected="setTagValue([tag.tag, 1], $event)"
							:clear-button="true"
						/>
					</div>
				</div>
				<!-- }}} -->
				<!-- type='checkboxes' {{{ -->
				<div v-else-if="tag.type == 'checkboxes'">
					<div v-for="option in tag.options" :key="option.id" class="form-check mr-3">
						<input
							class="form-check-input"
							type="checkbox"
							:id="`${tag.tag}-${option.id}`"
							:checked="!!tagValues[tag.tag][option.id]"
							@change="setTagValue([tag.tag, option.id], $event.target.checked)"
						/>
						<label
							class="form-check-label"
							:for="`${tag.tag}-${option.id}`"
						>
							{{option.title}}
						</label>
					</div>
				</div>
				<!-- }}} -->
				<!-- type='radios' {{{ -->
				<div v-else-if="tag.type == 'radios'">
					<div v-for="option in tag.options" :key="option.id" class="form-check mr-3">
						<input
							class="form-check-input"
							type="radio"
							:id="`${tag.tag}-${option.id}`"
							:name="tag.tag"
							:checked="tagValues[tag.tag] == option.id"
							@change="setTagValue(tag.tag, option.id)"
						/>
						<label
							class="form-check-label"
							:for="`${tag.tag}-${option.id}`"
						>
							{{option.title}}
						</label>
					</div>
				</div>
				<!-- }}} -->
				<!-- type unknown {{{ -->
				<div v-else class="alert alert-danger">
					Unknown search tag type "{{tag.type}}"
				</div>
				<!-- }}} -->
			</div>
		</div>
	</div>
</template>

<style lang="scss">
.search-input-tags {

}
</style>
