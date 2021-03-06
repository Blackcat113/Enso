<template>

    <div>
        <div class="columns"
            v-if="!summary">
            <div class="column is-4-desktop is-9-tablet is-12-mobile">
                <div class="">
                    <vue-select v-model="importType"
                        :options="importTypes"
                        @input="getTemplate"/>
                </div>
            </div>
            <div class="column is-hidden-touch is-6 has-padding-medium has-text-centered"
                v-if="importType">
                <file-uploader class="animated fadeIn"
                    :url="templateLink"
                    file-key="template"
                    @upload-start="loadingTemplate=true"
                    @upload-successful="template = $event;loadingTemplate = false"
                    @upload-error="loadingTemplate=false"
                    v-if="!template">
                    <a slot="upload-button"
                        class="button is-info"
                        slot-scope="{ openFileBrowser }"
                        @click="openFileBrowser">
                        <span>{{ __('Upload Template') }}</span>
                        <span class="icon is-small">
                            <fa icon="upload"/>
                        </span>
                    </a>
                </file-uploader>
                <a class="button is-info animated fadeIn has-margin-right-small"
                    v-if="template"
                    v-tooltip="template.original_name"
                    :href="downloadLink">
                    <span>{{ __('Download Template') }}</span>
                    <span class="icon is-small">
                        <fa icon="download"/>
                    </span>
                </a>
                <a class="button is-danger animated fadeIn"
                    @click="modal = true"
                    v-if="template"
                    :disabled="!canAccess('import.deleteTemplate')">
                    <span>{{ __('Delete Template') }}</span>
                    <span class="icon is-small">
                        <fa icon="trash-alt"/>
                    </span>
                </a>
            </div>
            <div class="column has-padding-medium has-text-centered"
                v-if="importType">
                <div class="">
                    <file-uploader :url="importLink"
                        file-key="import"
                        @upload-start=" importing = true"
                        @upload-successful="summary = $event;importing = false"
                        @upload-error="importing = false;importType = null"
                        v-if="importType">
                        <a slot="upload-button"
                            slot-scope="{ openFileBrowser }"
                            @click="openFileBrowser"
                            :class="['button is-success', { 'is-loading': importing }]">
                            <span>{{ __('Import') }}</span>
                            <span class="icon is-small">
                                <fa icon="upload"/>
                            </span>
                        </a>
                    </file-uploader>
                </div>
            </div>
        </div>
        <vue-table class="is-raised is-rounded animated fadeIn"
            :path="path"
            id="imports-table"
            @get-summary="getSummary"
            v-if="!summary">
            <b slot="successful"
            slot-scope="{ row }"
            class="has-text-success">
                {{ row['successful'] }}
            </b>
            <b slot="issues"
            slot-scope="{ row }"
            class="has-text-danger">
                {{ row['issues'] }}
            </b>
        </vue-table>
        <div class="columns"
            v-if="summary">
            <div class="column is-half-tablet is-one-third-widescreen is-one-quarter-fullhd">
                <nav class="box panel is-paddingless  animated bounceInLeft">
                    <p class="panel-heading">
                        {{ __('Import Summary') }}
                    </p>
                    <a class="panel-block">
                        <span class="panel-icon has-text-info">
                            <fa icon="file-excel"/>
                        </span>
                        {{ __('File') }}:&emsp;
                        <span class="has-text-info">
                            {{ summary.filename }}
                        </span>
                    </a>
                    <a class="panel-block">
                        <span class="panel-icon has-text-info">
                            <fa icon="calendar-alt"/>
                        </span>
                        {{ __('Date') }}:&emsp;
                        <span class="has-text-info">
                            {{ summary.date }}
                        </span>
                    </a>
                    <a class="panel-block">
                        <span class="panel-icon has-text-info">
                            <fa icon="clock"/>
                        </span>
                        {{ __('Time') }}:&emsp;
                        <span class="has-text-info">
                            {{ summary.time }}
                        </span>
                    </a>
                    <a class="panel-block">
                        <span class="panel-icon has-text-success">
                            <fa icon="check"/>
                        </span>
                        {{ __('Imported Entries') }}:&emsp;
                        <span class="has-text-success">
                            {{ summary.successful }}
                        </span>
                    </a>
                    <a class="panel-block">
                        <span class="panel-icon has-text-danger">
                            <fa icon="times"/>
                        </span>
                        {{ __('Issues') }}:&emsp;
                        <span class="has-text-danger">
                            {{ summary.issues }}
                        </span>
                    </a>
                    <div class="panel-block">
                        <button class="button is-info is-outlined is-fullwidth"
                            @click="summary=null">
                            {{ __("Back") }}
                        </button>
                    </div>
                </nav>
            </div>
            <div class="column is-half-tablet is-two-thirds-widescreen is-three-quarters-fullhd">
                <card class="animated bounceInRight"
                    :icon="icon"
                    :title="__('Issues')">
                    <tabs class="has-padding-medium"
                        alignment="right">
                        <tab v-for="(issues, category) in summary.structureIssues"
                            :key="category"
                            :id="category">
                            <ul class="issues has-margin-left-large">
                                <li v-for="(issue, index) in issues"
                                    :key="index">
                                    <span>
                                        <b class="has-text-danger">{{ issue }}</b>
                                    </span>
                                </li>
                            </ul>
                        </tab>
                        <tab v-for="(sheetIssues, sheet) in summary.contentIssues"
                            :key="sheet"
                            :id="sheet">
                            <tabs>
                                <tab v-for="(issues, category) in sheetIssues"
                                    :key="category"
                                    :id="category">
                                    <paginate :list="issues">
                                        <template slot-scope="{ list }">
                                            <h5 class="title is-5 has-text-centered">
                                                {{ __('Issues') }}
                                            </h5>
                                            <ul class="issues has-margin-left-large">
                                                <li v-for="(issue, index) in list"
                                                    :key="index">
                                                    <span v-if="issue.column">
                                                        {{ __("Column") }}:
                                                        <b class="has-text-warning">
                                                            {{ issue.column }}
                                                        </b>
                                                    </span>
                                                    <span v-if="issue.rowNumber">
                                                        {{ __("Line") }}:
                                                        <b class="has-text-warning">
                                                            {{ issue.rowNumber }}
                                                        </b>
                                                    </span>
                                                    <span v-if="issue.value">
                                                        {{ __("Value") }}:
                                                        <b class="has-text-danger">
                                                            {{ issue.value }}
                                                        </b>
                                                    </span>
                                                </li>
                                            </ul>
                                        </template>
                                    </paginate>
                                </tab>
                            </tabs>
                        </tab>
                    </tabs>
                </card>
            </div>
        </div>
        <modal :show="modal"
            @close="modal = false"
            :i18n="__"
            @commit="deleteTemplate(template.id); modal = false"/>
    </div>

</template>

<script>

import { VTooltip } from 'v-tooltip';
import { library } from '@fortawesome/fontawesome-svg-core';
import {
    faUpload, faDownload, faTrashAlt, faFileExcel,
    faCalendarAlt, faClock, faCheck, faTimes, faBook,
} from '@fortawesome/free-solid-svg-icons';
import VueSelect from '../../components/enso/select/VueSelect.vue';
import VueTable from '../../components/enso/vuedatatable/VueTable.vue';
import FileUploader from '../../components/enso/fileuploader/FileUploader.vue';
import Modal from './Modal.vue';
import Card from '../../components/enso/bulma/Card.vue';
import Overlay from '../../components/enso/bulma/Overlay.vue';
import Paginate from '../../components/enso/bulma/Paginate.vue';
import Tabs from '../../components/enso/bulma/Tabs.vue';
import Tab from '../../components/enso/bulma/Tab.vue';

library.add([
    faUpload, faDownload, faTrashAlt, faFileExcel,
    faCalendarAlt, faClock, faCheck, faTimes, faBook,
]);

export default {
    components: {
        VueSelect, VueTable, FileUploader, Card, Modal, Overlay, Tabs, Tab, Paginate,
    },

    directives: { tooltip: VTooltip },

    data() {
        return {
            path: route('import.initTable'),
            importType: null,
            summary: null,
            template: null,
            modal: false,
            loadingTemplate: false,
            importing: false,
            importTypes: [],
        };
    },

    computed: {
        templateLink() {
            return this.importType
                && route('import.uploadTemplate', this.importType);
        },
        downloadLink() {
            return this.template
                && route('import.downloadTemplate', this.template.id);
        },
        importLink() {
            return this.importType
                && route('import.run', this.importType);
        },
        icon() {
            return faBook;
        },
    },

    created() {
        axios.get(route('import.index'))
            .then(({ data }) => {
                this.importTypes = data.importTypes;
            }).catch(error => this.handleError(error));
    },

    methods: {
        getTemplate() {
            if (!this.importType) {
                return;
            }

            this.loadingTemplate = true;

            axios.get(route('import.getTemplate', this.importType))
                .then(({ data }) => {
                    this.template = data;
                    this.loadingTemplate = false;
                }).catch((error) => {
                    this.loadingTemplate = false;
                    this.handleError(error);
                });
        },
        deleteTemplate(id) {
            this.loadingTemplate = true;
            axios.delete(route('import.deleteTemplate', id))
                .then(({ data }) => {
                    this.template = null;
                    this.modal = false;
                    this.$toastr.success(data.message);
                    this.loadingTemplate = false;
                }).catch((error) => {
                    this.modal = false;
                    this.loadingTemplate = false;
                    this.handleError(error);
                });
        },
        getSummary(row) {
            this.loading = true;

            axios.get(route('import.getSummary', row.dtRowId))
                .then(({ data }) => {
                    this.loading = false;

                    if (data.issues === 0) {
                        this.$toastr.info('The import has no issues');
                        return;
                    }

                    this.summary = data;
                }).catch(error => this.handleError(error));
        },
    },
};

</script>

<style lang="scss" scoped>

    ul.issues {
        list-style-type: square;

        li {
            cursor: pointer;
            width: fit-content;
            padding: 0.2em;

            &:hover {
                background: lightgray;
            }
        }
    }

</style>
