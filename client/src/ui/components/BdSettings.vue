/**
 * BetterDiscord Settings Component
 * Copyright (c) 2015-present Jiiks/JsSucks - https://github.com/Jiiks / https://github.com/JsSucks
 * All rights reserved.
 * https://betterdiscord.net
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
*/

<template>
    <div class="bd-settings" :class="{active, 'bd-settings-out': !item && animating}" @keyup="$emit('close')">
        <SidebarView :contentVisible="item" :animating="animating" :class="{'bd-stop': item}">
            <Sidebar slot="sidebar">
                <div class="bd-settings-x" @click="$emit('close')">
                    <MiClose size="17"/>
                    <span class="bd-x-text">ESC</span>
                </div>
                <template v-for="(category, text) in sidebar">
                    <SidebarItem :item="{text, type: 'header'}" />
                    <SidebarItem v-for="i in category" :key="i.id" :item="i" :active="item && i.id === item.id" @click="itemOnClick(i.id)" />
                </template>
            </Sidebar>

            <div slot="sidebarfooter" class="bd-info">
                <span class="bd-vtext">v2.0.0a by Jiiks/JsSucks</span>
                <div @click="openGithub" v-tooltip="'GitHub'" class="bd-material-button">
                    <MiGithubCircle size="16" />
                </div>
                <div @click="openTwitter" v-tooltip="'@Jiiksi'" class="bd-material-button">
                    <MiTwitterCircle size="16" />
                </div>
                <div @click="openWebsite" v-tooltip="'BetterDiscord'" class="bd-material-button">
                    <MiWeb size="16" />
                </div>
            </div>

            <ContentColumn slot="content">
                <transition name="bd-contentcolumn" @before-enter="animating++" @after-enter="animating--" @enter-cancelled="animating--" @before-leave="animating++" @after-leave="animating--" @leave-cancelled="animating--">
                    <div v-if="item" :key="item.id">
                        <template v-if="item.component">
                            <component :is="item.component" :SettingsWrapper="SettingsWrapper" />
                        </template>

                        <SettingsWrapper v-else-if="typeof item.set === 'string'" :headertext="Settings.getSet(item.set).headertext">
                            <SettingsPanel :settings="Settings.getSet(item.set)" :schemes="Settings.getSet(item.set).schemes" />
                        </SettingsWrapper>
                        <SettingsWrapper v-else-if="item.set" :headertext="item.set.headertext">
                            <SettingsPanel :settings="item.set" :schemes="item.set.schemes" />
                        </SettingsWrapper>

                        <ConnectivityView v-else-if="item.contentid === 'connectivity'" />
                        <CssEditorView v-else-if="item.contentid === 'css'" />
                        <PluginsView v-else-if="item.contentid === 'plugins'" />
                        <ThemesView v-else-if="item.contentid === 'themes'" />
                        <UpdaterView v-else-if="item.contentid === 'updater'" />
                    </div>
                </transition>
            </ContentColumn>
        </SidebarView>
    </div>
</template>

<script>
    // Imports
    import { Events, Settings } from 'modules';
    import { BdMenuItems } from 'ui';
    import { shell } from 'electron';
    import { SidebarView, Sidebar, SidebarItem, ContentColumn } from './sidebar';
    import { SettingsWrapper, SettingsPanel, CssEditorView, PluginsView, ThemesView, UpdaterView, ConnectivityView } from './bd';
    import { SvgX, MiGithubCircle, MiWeb, MiClose, MiTwitterCircle } from './common';

    export default {
        data() {
            return {
                animating: 0,
                item: null,
                items: BdMenuItems.items,
                Settings,
                SettingsWrapper,
                openMenuHandler: null
            };
        },
        props: ['active'],
        components: {
            SidebarView, Sidebar, SidebarItem, ContentColumn,
            SettingsWrapper, SettingsPanel, CssEditorView, PluginsView, ThemesView, UpdaterView, ConnectivityView,
            MiGithubCircle, MiWeb, MiClose, MiTwitterCircle
        },
        computed: {
            sidebar() {
                const categories = {};
                for (let item of this.items) {
                    if (item.hidden) continue;
                    const category = categories[item.category] || (categories[item.category] = []);
                    category.push(item);
                }
                return categories;
            }
        },
        methods: {
            itemOnClick(id) {
                this.item = this.items.find(item => item.id === id);
            },
            closeContent() {
                this.item = null;
            },
            openGithub() {
                shell.openExternal('https://github.com/JsSucks/BetterDiscordApp');
            },
            openWebsite() {
                shell.openExternal('https://betterdiscord.net');
            },
            openTwitter() {
                shell.openExternal('https://twitter.com/Jiiksi');
            }
        },
        watch: {
            active(active) {
                if (active) return;
                this.closeContent();
            }
        },
        created() {
            Events.on('bd-open-menu', this.openMenuHandler = item => item && this.itemOnClick(this.items.find(i => i === item || i.id === item || i.contentid === item || i.set === item).id));
        },
        destroyed() {
            if (this.openMenuHandler) Events.off('bd-open-menu', this.openMenuHandler);
        }
    }
</script>
