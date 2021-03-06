<template>
    <v-dialog max-width="600" v-model="open">
        <v-card>
            <v-card-title class="pa-1">
                <v-layout class="text-center">
                    <v-flex>Settings</v-flex>
                </v-layout>
                <v-btn color="neutral"
                       @click="open = false"
                       small icon absolute right>
                    <v-icon small>mdi-close</v-icon>
                </v-btn>
            </v-card-title>
            <v-divider light />
            <v-card-text class="pa-0 settings-content relative">
                <v-layout class="absolute-expand">
                    <v-flex xs3>
                        <v-list class="pt-0 fill-height" color="controls">
                            <v-list-item-group v-model="selected" mandatory>
                                <v-list-item v-for="page in pages" :key="page">
                                    <v-list-item-content>
                                        <v-list-item-title>{{ page }}</v-list-item-title>
                                    </v-list-item-content>
                                </v-list-item>
                            </v-list-item-group>
                        </v-list>
                    </v-flex>
                    <v-flex class="panel px-5">
                        <v-container v-if="selected === 0">
                            <v-radio-group v-model="settings.dark_mode" row>
                                <v-radio label="Light" value="light" on-icon="mdi-white-balance-sunny" />
                                <v-spacer />
                                <v-radio label="Dark" value="dark" on-icon="mdi-weather-night" />
                                <v-spacer />
                                <v-radio v-if="supportsThemeDetection" label="Automatic" value="auto" on-icon="mdi-theme-light-dark" />
                            </v-radio-group>
                        </v-container>
                        <v-container v-if="selected === 1">
                            <v-switch v-model="settings.network.sleep" label="Disconnect when Network is not selected" hide-details />
                            <v-flex offset-xs1>
                                <small>(This helps to prevent unnecessary memory and cpu usage)</small>
                            </v-flex>
                            <v-switch v-model="settings.network.sort_params" label="Sort request parameters" hide-details />
                            <v-row class="mt-2">
                                <v-col cols="3">
                                    <v-text-field
                                            label="Port"
                                            v-model="settings.port"
                                            v-mask="'#####'"
                                            hide-details />
                                </v-col>
                                <v-col cols="4">
                                    <v-text-field
                                            label="Clear requests after"
                                            v-model="settings.network.limit"
                                            v-mask="'#####'"
                                            hide-details />
                                </v-col>
                            </v-row>
                            <v-row>
                                <v-col>
                                    <v-text-field
                                            label="Adapter blacklist"
                                            v-model="settings.adapter_blacklist"
                                            hide-details />
                                </v-col>
                            </v-row>
                        </v-container>
                    </v-flex>
                </v-layout>
            </v-card-text>
        </v-card>
    </v-dialog>
</template>

<script>
    import { darkModeMatcher } from './utils'

    export default {
        name: 'Settings',
        props: {
            value: Boolean,
            settings: Object
        },
        data: () => ({
            selected: 0,
            pages: ['Theme', 'Network']
        }),
        computed: {
            open: {
                get () {
                    return this.value
                },
                set (value) {
                    this.$emit('input', value)
                }
            },
            supportsThemeDetection () {
                return !!darkModeMatcher
            }
        },
        watch: {
            'settings.dark_mode': {
                handler (value) {
                    if (value === 'auto') {
                        this.dark_mode = darkModeMatcher.matches
                    } else {
                        this.dark_mode = value === 'dark'
                    }
                },
                immediate: true
            }
        },
        mounted () {
            darkModeMatcher?.addEventListener('change', this.darkModeObserver)
        },
        beforeDestroy () {
            darkModeMatcher?.removeEventListener('change', this.darkModeObserver)
        },
        methods: {
            darkModeObserver (e) {
                if (this.settings.dark_mode === 'auto') {
                    this.dark_mode = e.matches
                }
            }
        }
    }
</script>

<style lang="scss" scoped>
    .settings-content {
        min-height: 400px;
    }
</style>
