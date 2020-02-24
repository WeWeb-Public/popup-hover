<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div v-if="display" class="popup-hover">
        <!-- Weweb Wallpaper -->
        <wwObject class="background" v-bind:ww-object="section.data.color" ww-category="background"></wwObject>

        <div class="popup-container">
            <!-- wwManager:start -->
            <wwSectionEditMenu v-bind:sectionCtrl="sectionCtrl" :options="openOptions" smaal ></wwSectionEditMenu>
            <!-- wwManager:end -->
            <wwObject class="background" v-bind:ww-object="section.data.background" ww-category="background"></wwObject>
            <wwObject class="close" v-bind:ww-object="section.data.close" @click="close"></wwObject>
            <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="section.data.list" class="list" @ww-add="add(section.data.list, $event)" @ww-remove="remove(section.data.list, $event)">
                <wwObject tag="div" v-for="object in section.data.list" :key="object.uniqueId" :ww-object="object"></wwObject>
            </wwLayoutColumn>
            <wwObject class="close-text" v-bind:ww-object="section.data.closeText" @click="close"></wwObject>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>
export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            display: false
        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        }
    },
    created() {
        this.init()

    },
    mounted() {
        const alreadySeen = this.$cookies.get('popup-hover-closed') || false;
        if (wwLib.manager) {
            this.tictacBeforeDisplay()

        } else if (!alreadySeen) {
            this.tictacBeforeDisplay()
        }
    },
    destroyed() {
        this.toggleBodyClass(false, 'overflow-hidden')
    },
    methods: {
        init() {
            //Initialize section data
            let needUpdate = false

            this.section.data = this.section.data || {};
            this.section.data.timeBeforeDisplaying = this.section.data.timeBeforeDisplaying || 3

            if (!this.section.data.color) {
                this.section.data.color = wwLib.wwObject.getDefault({
                    type: 'ww-color'
                });
                needUpdate = true
            }

            if (!this.section.data.background) {
                this.section.data.background = wwLib.wwObject.getDefault({
                    type: 'ww-color'
                });
                needUpdate = true
            }

            if (!this.section.data.list) {
                this.section.data.list = [];
                needUpdate = true;
            }

            if (!this.section.data.close) {
                this.section.data.close = wwLib.wwObject.getDefault({
                    type: 'ww-icon'
                });
                needUpdate = true
            }

            if (!this.section.data.closeText) {
                this.section.data.closeText = wwLib.wwObject.getDefault({
                    type: 'ww-text'
                });
                needUpdate = true
            }

            if (needUpdate) {
                this.sectionCtrl.update(this.section);
            }
        },
        tictacBeforeDisplay() {
            setTimeout(() => {
                this.addCookie()
                this.display = true;
                this.toggleBodyClass(true, 'overflow-hidden');
            }, this.section.data.timeBeforeDisplaying * 1000);
        },
        addCookie() {
            // Remove cookie
            // this.$cookies.remove('popup-hover-closed');
            // Set in cookies the isCookieAgreed for 24h
            var in24h = new Date();
            in24h.setDate(in24h.getDate() + 1);
            this.$cookies.set('popup-hover-closed', true, { 'expires': in24h, path: '/' });
        },
        toggleBodyClass(add, className) {
            const el = document.documentElement;
            if (add) {
                el.classList.add(className);
            } else {
                el.classList.remove(className);
            }
        },
        close() {
            if (this.editMode) return;
            // reactivate body scroll
            this.toggleBodyClass(false, 'overflow-hidden');
            this.display = false;
        },
        /* wwManager:start */
        add(list, options) {
            try {
                list.splice(options.index, 0, options.wwObject);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        async openOptions() {
            try {
                wwLib.wwPopups.addStory('POPUPHOVER_CONFIG', {
                    title: {
                        en: 'Popup configuration',
                        fr: 'Configuration du popup'
                    },
                    type: 'wwPopupForm',
                    storyData: {
                        fields: [
                            {
                                label: {
                                    en: 'Time before displaying the popup (in second):',
                                    fr: 'Temps avant l\'apparition du popup (en secondes) :'
                                },
                                type: 'text',
                                key: 'timeBeforeDisplaying',
                                valueData: 'section.data.timeBeforeDisplaying'
                            }
                        ]
                    },
                    buttons: {
                        NEXT: {
                            text: {
                                en: 'Finish',
                                fr: 'Terminer'
                            },
                            next: null
                        }
                    }
                })
                let options = {
                    firstPage: 'POPUPHOVER_CONFIG',
                    data: {
                        section: this.section,
                    },
                }
                const result = await wwLib.wwPopups.open(options)
                let needUpdate = false
                if (typeof (result) != 'undefined') {
                    if (typeof (result.timeBeforeDisplaying) != 'undefined') {
                        this.section.data.timeBeforeDisplaying = result.timeBeforeDisplaying
                        needUpdate = true
                    }
                    if (needUpdate) {
                        this.sectionCtrl.update(this.section);
                        this.$forceUpdate();
                    }
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* wwManager:end */

    }
}
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang="scss" scoped>
.popup-hover {
    position: fixed;
    top: 0;
    height: 100vh;
    left: 0;
    right: 0;
    z-index: 100;
    .background {
        position: absolute;
        top: 0;
        left: 0;
        height: 100%;
        width: 100%;
    }
    .popup-container {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        min-height: 50px;
        width: 90%;
        box-shadow: 0 2px 50px 0 rgba(0, 0, 0, 0.2);
        @media (min-width: 768px) {
            width: 60%;
        }
        @media (min-width: 992px) {
            width: 40%;
        }
        @media (min-width: 1200px) {
            width: 30%;
        }
        .list {
            height: 100%;
            width: 100%;
        }
    }
    .close {
        position: absolute;
        top: 20px;
        right: 20px;
        padding: 20px;
        z-index: 21;
        cursor: pointer;
    }
    .close-text {
        cursor: pointer;
    }
}
</style>
<style>
.overflow-hidden {
    overflow: hidden;
}
</style>

