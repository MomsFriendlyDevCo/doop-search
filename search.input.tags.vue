<script lang="js" frontend>
import Debug from '@doop/debug';

const $debug = Debug('@doop/search').enable(true);

/**
* TODO: Docs
*/
app.component('searchInputTags', {
	data() { return {
		tagValues: {}, // Lookup values for each tag
		computedTags: [], // Cleaned up version of $props.tags via compileTags()
	}},
	props: {
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
		* Compile tag spec + populate tagValue defaults
		* This function mutates computedTags + tagValues to suitable defaults
		*/
		compileTags() {
			$debug('compileTags', this.$props.tags);
			var tagValues = {}; // Holder for new tagValues (default values of widgets)

			this.computedTags = this.$props.tags.map((tag, tagIndex) => {
				if (!tag.tag) throw new Error(`No tag property specified for search tag index #${tagIndex}`);
				if (!tag.title) throw new Error(`No title property specified for search tag "${tag.tag}"`);
				if (!tag.type) throw new Error(`No type property specified for search tag "${tag.tag}"`);

				switch (tag.type) {
					case 'hidden':
						tagValues[tag.tag] = tag.default;
						break;
					case 'date':
						tagValues[tag.tag] = tag.default;
						if (!tag.dateFormat) tag.dateFormat = 'DD/MM/YYYY';
						break;
					case 'dateRange':
						tagValues[tag.tag] = [undefined, undefined];
						if (!tag.dateFormat) tag.dateFormat = 'DD/MM/YYYY';
						if (!tag.seperator) tag.seperator = '-';
						if (!tag.toQuery) tag.toQuery = tag => // Special dataRange default output function
							_.isEmpty(tag.value) ? null // Uninitalized dates
							: !tag.value[0] && !tag.value[1] ? null // No dates
							: tag.value[0] && tag.value[1] ? // Has both start + end
								tag.tag + ':'
								+ moment(tag.value[0]).format(tag.dateFormat)
								+ tag.seperator
								+ moment(tag.value[1]).format(tag.dateFormat)
							: tag.value[0] && tag.startOnlyTag ? // Only has start + alternate starting tag
								tag.startOnlyTag + ':' + moment(tag.value[0]).format(tag.dateFormat)
							: tag.value[0] ? // Only has start
								tag.tag + ':' + moment(tag.value[0]).format(tag.dateFormat) + tag.seperator
							: tag.value[1] && tag.endOnlyTag ? // Only has end + alternate ending tag
								tag.endOnlyTag + ':' + moment(tag.value[1]).format(tag.dateFormat)
							: tag.value[1] ? // Only has end
								tag.tag + ':' + tag.seperator + moment(tag.value[1]).format(tag.dateFormat)
							: null; // All other cases
						break;
					case 'digest':
						if (!tag.digest || !_.isPlainObject(tag.digest)) throw new Error(`No digest config specified for digest search tag "${tag.tag}"`);
						tag.digest = { // Set suitable digest values
							allowRemove: true,
							class: 'col-sm-9 form-control-plaintext',
							classValid: 'badge badge-primary',
							classInvalid: 'badge badge-danger',
							lazy: false,
							...tag.digest,
						};
						tagValues[tag.tag] = tag.default;
						break;
					case 'checkboxes':
					case 'radios':
						if (!tag.options || !_.isArray(tag.options)) throw new Error(`No options config specified for checkboxes / radio search tag "${tag.tag}"`);
						tag.options = tag.options.map(option => {
							if (_.isString(option)) {
								return {id: option, title: _.startCase(option)}; // Transform string into iterable {id, title}
							} else if (_.isObject(option)) {
								if (!option.id || !option.title) throw new Error(`Search tag "${tag.tag}" options must be of the form {id: String, title: String} if specifying object values`);
								return option;
							} else {
								throw new Error(`Unknown option type for checkbox / radio search tag "${tag.tag}"`);
							}
						});

						if (tag.type == 'radios') {
							if (!tag.clearValue) tag.clearValue = tag.options[0].id; // Allocate clearValue if its a radio
							if (!tag.default) tag.default = tag.options[0].id;
							tagValues[tag.tag] = tag.default;
						} else if (tag.type == 'checkboxes') {
							tagValues[tag.tag] = _(tag.options).mapKeys(v => v.id).mapValues(v => false).value();
						}
						break;
					default:
						throw new Error(`Unknown search tag type tag "${tag.tag}"`);
				}

				if (!tag.toQuery) tag.toQuery = tag => tag.value ? `${tag.tag}:${tag.value}` : false;

				return tag;
			});

			// FIXME: Can we assign directly within logic above?
			_.forEach(tagValues, (v, k) => this.$set(this.tagValues, k, v)); // Assign Vue managed tagValues to the defaults we allocated above

			$debug('computedTags', this.computedTags);
		},


		/**
		* Compute local state into a search query (also set the search query display)
		*/
		encodeQuery() {
			$debug('encodeQuery', this.computedTags);

			// TODO: Each input control must provide it's part of the query for concatination

			// Precalculate tags so we know if any have been defined
			const tags = this.computedTags // Computed tag entries
				.map(tag => ({...tag})) // Clone tag so further mutations don't effect source
				.map(tag => { // Add tag.value, if any
					switch (tag.type) {
						case 'digest': // Simple key=val setters
							tag.value = this.tagValues[tag.tag];
							break;
						case 'hidden':
							tag.value = undefined;
							break;
						case 'radios':
							tag.value = this.tagValues[tag.tag] && this.tagValues[tag.tag] != tag.clearValue && this.tagValues[tag.tag];
							break;
						case 'date':
							tag.rawValue = this.tagValues[tag.tag]; // Add raw full date in case its needed
							tag.value = this.tagValues[tag.tag] && moment(this.tagValues[tag.tag]).format(tag.dateFormat);
							break;
						case 'dateRange':
							tag.value = this.tagValues[tag.tag];
							break;
						case 'checkboxes':
							var values = _.chain(this.tagValues[tag.tag])
								.pickBy() // Choose only truthy values
								.keys()
								.value();

							tag.value = values.length ? values.join(',') : false;
							break;
						default:
							tag.value = false;
							console.warn(`Cant convert unknown tag type "${tag.type}" to query string - skipping`);
					}

					return tag;
				})
				.map(tag => tag.toQuery(tag)) // Call toQuery on each object expecting either a value or falsy
				.filter(Boolean)
				.join(' ')
				.trim();

			;
			// TODO: Report query via event?
			$debug('encodeQuery', tags);
		},


		/**
		* Decode a string query into local settings
		* This function mutates `tagValues` to match the incomming queryString + `fuzzyQuery` with human specified parts
		* @param {string} query String query to decode back into its component parts
		*/
		decodeQuery(query) {
			$debug('decodeQuery', query);

			this.computedTags.forEach(tag => {
				switch (tag.type) {
					case 'hidden': // Simple key / values
					case 'digest':
						this.tagValues[tag.tag] = queryHash[tag.tag];
						break;
					case 'date':
						this.tagValues[tag.tag] = queryHash[tag.tag] ? moment(queryHash[tag.tag], tag.dateFormat).toDate() : undefined;
						break;
					case 'dateRange':
						if (_.isEmpty(queryHash[tag.tag])) {
							this.tagValues[tag.tag] = [undefined, undefined];
						} else {
							var dateSplit = queryHash[tag.tag].split(/\s*-\s*/, 2);
							this.tagValues[tag.tag] = [
								dateSplit[0] ? moment(dateSplit[0], tag.dateFormat).toDate() : undefined,
								dateSplit[1] ? moment(dateSplit[1], tag.dateFormat).toDate() : undefined,
							];
						}
						break;
					case 'checkboxes':
						var setValues = new Set((queryHash[tag.tag] || '').split(/\s*,\s*/).filter(Boolean));
						this.tagValues[tag.tag] = _(tag.options)
							.mapKeys(option => option.id)
							.mapValues(option => setValues.has(option.id))
							.value()
						break;
					case 'radios':
						this.tagValues[tag.tag] = queryHash[tag.tag];
						break;
				}
			});
			$debug('tagValues', this.tagValues);
		},


		/**
		* Compute and return the properties of a digest component
		* @param {Object} tag The tag being exmined
		* @returns {Object} The computed digestSelect widget properties
		*/
		widgetDigestProps(tag) {
			$debug('widgetDigestProps', tag);
			return {
				selected: this.tagValues[tag.tag],
				...tag.digest, // Merge in remainder of tag.digest options
			};
		},
	},

	created() {
		$debug('created');
	},

	watch: {
		tags: { // React to tag definition changes
			immediate: true,
			handler(newVal, oldVal) {
				$debug('$watch', 'tags', newVal, oldVal);
				if (JSON.stringify(newVal) == JSON.stringify(oldVal)) return; // Horrible kludge to detect if tag composition is identical - if so skip rebuild
				this.compileTags();
			},
		},
	},
});
</script>

<template>
	<div class="search-input-tags">
		<div v-for="tag in computedTags.filter(ct => ct.type != 'hidden')" :key="tag.tag" class="form-group">
			<label class="col-sm-3 col-form-label">{{tag.title}}</label>
			<div class="col-9 mt-2">
				<!-- type='digest' {{{ -->
				<dynamic-component
					v-if="tag.type == 'digest'"
					component="digestSelect"
					:props="widgetDigestProps(tag)"
					:events="{change: setTagValue.bind(null, tag.tag)}"
				/>
				<!-- }}} -->
				<!-- type='date' {{{ -->
				<v-date
					v-else-if="tag.type == 'date'"
					:value="tagValues[tag.tag]"
					@selected="setTagValue(tag.tag, $event)"
					:clear-button="true"
				/>
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
