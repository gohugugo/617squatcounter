<template>
    <v-app>
        <v-app-bar app dark color="primary">
            <v-row align="center" justify="space-between">
                <!--               Title                -->
                <v-col>
                    <h3>Labeliser</h3>
                </v-col>

                <!--                Number of labelised images              -->
                <v-col align="center">
                    <header> {{nbLabelised}}/{{nbImages}} labelised</header>
                </v-col>

                <v-col align="end">

                    <!--                    Individual/Multiple Mode                -->
                    <v-btn icon @click="modeIndiv=!modeIndiv">
                        <v-icon> {{!modeIndiv ? "mdi-image" : "mdi-image-multiple"}}</v-icon>
                    </v-btn>

                    <!--                    Menu                -->
                    <v-menu offset-y>
                        <template v-slot:activator="{ on }">
                            <v-btn icon v-on="on">
                                <v-icon>mdi-menu</v-icon>
                            </v-btn>
                        </template>

                        <v-card class="px-2">
                            <v-list dense nav class="mr-2">

                                <v-subheader align="end"> Images</v-subheader>
                                <v-divider class="pa-0"/>

                                <v-list-item link class="pr-5" @click="uploadClicked">
                                    <v-list-item-icon>
                                        <v-icon>mdi-download</v-icon>
                                    </v-list-item-icon>
                                    <v-list-item-content>
                                        <v-list-item-title>Import</v-list-item-title>
                                        <input
                                                ref="uploader"
                                                class="d-none"
                                                type="file"
                                                accept="text/csv/"
                                                @change="imagesFilesChanged"
                                                multiple
                                        >
                                    </v-list-item-content>
                                </v-list-item>

                                <v-subheader align="end"> Labels</v-subheader>
                                <v-divider class="pa-0"/>

                                <v-list-item link class="pr-5" @click="importClicked">
                                    <v-list-item-icon>
                                        <v-icon>mdi-download</v-icon>
                                    </v-list-item-icon>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Import
                                        </v-list-item-title>
                                        <input
                                                ref="importer"
                                                class="d-none"
                                                type="file"
                                                accept="text/csv/"
                                                @change="labelsFileChanged"
                                        >
                                    </v-list-item-content>
                                </v-list-item>

                                <v-list-item link class="pr-5" @click="exportCsv">
                                    <v-list-item-icon>
                                        <v-icon>mdi-upload</v-icon>
                                    </v-list-item-icon>
                                    <v-list-item-content>
                                        <v-list-item-title>Export</v-list-item-title>
                                    </v-list-item-content>
                                </v-list-item>

                                <v-list-item link class="pr-5" @click.stop="dialog=true">
                                    <v-list-item-icon>
                                        <v-icon>mdi-eye</v-icon>
                                    </v-list-item-icon>
                                    <v-list-item-content>
                                        <v-list-item-title>
                                            Open
                                        </v-list-item-title>
                                    </v-list-item-content>
                                </v-list-item>
                            </v-list>
                        </v-card>
                    </v-menu>
                </v-col>

            </v-row>
        </v-app-bar>

        <v-main>
            <transition name="fade" mode="out-in">
                <v-container fill-height>

                    <!--                    Dialog window to see labels             -->
                    <v-dialog
                            v-model="dialog"
                            hide-overlay
                            fullscreen
                            transition="dialog-bottom-transition"
                    >
                        <v-card>
                            <v-toolbar
                                    dark
                                    color="primary"
                            >
                                <v-btn
                                        icon
                                        dark
                                        @click="dialog = false"
                                >
                                    <v-icon>mdi-close</v-icon>
                                </v-btn>
                                <v-toolbar-title>Labels</v-toolbar-title>
                                <v-spacer/>
                                <v-toolbar-items @click="clearLabels">
                                    <v-btn
                                            icon
                                    >
                                        <v-icon>mdi-refresh</v-icon>
                                    </v-btn>
                                    <v-btn
                                            icon
                                            @click="exportCsv"
                                    >
                                        <v-icon>mdi-upload</v-icon>
                                    </v-btn>
                                </v-toolbar-items>
                            </v-toolbar>
                            <v-container>
                                <v-row justify="center">
                                    <v-col cols="6">
                                        <v-data-table
                                                :headers="headers"
                                                :items="labelsTable"
                                                :items-per-page="10"
                                                class="elevation-1"
                                        ></v-data-table>
                                    </v-col>
                                </v-row>
                            </v-container>

                        </v-card>
                    </v-dialog>

                    <!--                    Import image button when no images              -->
                    <v-row
                            class="text-center justify-center"
                            v-if="nbImages===0"
                    >
                        <v-col cols="5">
                            <v-btn text @click="uploadClicked">
                                Import images
                            </v-btn>
                            <input
                                    ref="uploader"
                                    class="d-none"
                                    type="file"
                                    accept="text/csv/"
                                    @change="imagesFilesChanged"
                                    multiple
                            >
                        </v-col>

                    </v-row>

                    <!--                    Individual Mode             -->
                    <v-row class="text-center" v-if="modeIndiv && (nbImages!==0)" height="50%" row="5">

                        <v-col :cols="portrait ? 12 : 7" >
                            <v-img
                                    :src="images[index].path"
                                    transition="fade-transition"
                                    max-height="400"
                                    contain
                                    class="elevation-2"
                                    @click="spacePressed"
                            />
                        </v-col>

                        <v-col justify="center">
                            <v-container fill-height>
                                <v-row>
                                    <v-col cols="12" class="pb-0">
                                        <h3 class="primary--text">Image nº {{index + 1}}</h3>
                                    </v-col>
                                    <v-col cols="12" class="pb-0">
                                        <h4 class="primary--text">Nom du fichier :</h4>
                                    </v-col>
                                    <v-col cols="12" class="pt-0">
                                        <header>{{image.filename}}</header>
                                    </v-col>
                                    <v-col cols="12" class="pb-0">
                                        <h4 class="primary--text">Label :</h4>
                                    </v-col>
                                    <v-col>
                                        <h1>{{this.count}}</h1>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-col>

                    </v-row>

                    <!--                    Multiple mode               -->
                    <v-row class="text-center justify-center" v-else>
                        <v-col
                                cols="md-2"
                                sm=5
                                v-for="(image,i) in images"
                                :key="i"
                                class="mx-2"
                        >
                            <v-row class="text-center justify-center">
                                <v-img
                                        :src="image.path"
                                        transition="fade-transition"
                                        max-height="400"
                                        contain
                                        class="elevation-2"
                                        @click="[index=i,modeIndiv=true]"
                                />
                            </v-row>

                            <v-row class="text-center justify-center">
                                <h1>{{labels[image.filename]}}</h1>
                            </v-row>
                        </v-col>
                    </v-row>

                </v-container>
            </transition>
        </v-main>

        <v-bottom-navigation app height="45" class="elevation-0" v-if="modeIndiv && (nbImages!==0)">
            <v-btn icon @click="leftPressed">
                <v-icon>mdi-chevron-left</v-icon>
            </v-btn>
            <v-btn icon @click="rightPressed">
                <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
            <v-spacer/>

            <v-btn icon @click="downPressed">
                <v-icon>mdi-refresh</v-icon>
            </v-btn>
            <v-spacer></v-spacer>
            <v-btn icon @click="enterPressed">
                <v-icon color="green">mdi-check</v-icon>
            </v-btn>
            <v-spacer></v-spacer>

        </v-bottom-navigation>
    </v-app>
</template>

<script>
    export default {
        name: 'App',

        data: () => ({
            touchDevice: false,
            dialog: false,
            images: [],
            labels: {},
            index: 0,
            count: undefined,
            modeIndiv: true,
            headers: [
                {
                    text: 'filename',
                    value: 'filename',
                },
                {text: 'Label', value: 'label', width: "10%"}],
            labelsTable: [],
        }),

        mounted() {
            this.initialiseKeyboardListener();
            this.loadLabels();
        },

        created() {
            if ("ontouchstart" in document.documentElement) {
                this.touchDevice = true;
            }
        },

        destroyed() {
        },

        methods: {
            initialiseKeyboardListener() {
                window.addEventListener("keydown", function (e) {
                    console.log(e);
                    if (this.modeIndiv && this.image) {
                        if (e.keyCode === 13) {
                            this.enterPressed();
                        }
                        if (e.keyCode === 32) {
                            this.spacePressed();
                        }
                        if (e.keyCode === 37) {
                            this.leftPressed();
                        }
                        if (e.keyCode === 39) {
                            this.rightPressed();
                        }
                        if (e.keyCode === 38) {
                            this.upPressed();
                        }
                        if (e.keyCode === 40) {
                            this.downPressed();
                        }
                    }
                }.bind(this));
            },

            loadLabels() {
                let labels = JSON.parse(localStorage.getItem('labels'));
                this.labels = labels ? labels : {};
                if (this.nbImages > 0) {
                    this.updateCount();
                }
            },

            exportCsv() {
                const rows = [["filename", "count"]];

                for (let image of this.images) {
                    rows.push([image.filename, this.labels[image.filename]]);
                }

                let csvContent = "data:text/csv;charset=utf-8," + rows.map(e => e.join(",")).join("\n");

                var encodedUri = encodeURI(csvContent);
                var link = document.createElement("a");
                link.setAttribute("href", encodedUri);
                link.setAttribute("download", "labels.csv");
                document.body.appendChild(link); // Required for FF

                link.click();
            },

            uploadClicked() {
                this.$refs.uploader.click()
            },

            imagesFilesChanged(e) {
                let images = [];
                let that = this;
                let uploadedImages = e.target.files;
                uploadedImages.forEach(f => {
                    let reader = new FileReader();
                    reader.onload = function (e) {
                        images.push({path: e.target.result, filename: f.name});
                        that.updateCount();
                    };
                    reader.readAsDataURL(f);
                });
                this.images = images;
                this.index = 0;
                this.updateCount();
            },

            importClicked() {
                this.$refs.importer.click()
            },

            labelsFileChanged(e) {
                let selectedFile = e.target.files[0];
                selectedFile.text().then(e => {
                    let bufferString = e;
                    let arr = bufferString.split('\n');
                    for (let i = 0; i < arr.length; i++) {
                        arr[i] = arr[i].split(",");
                    }
                    for (let i = 1; i < arr.length; i++) {
                        this.labels[arr[i][0]] = arr[i][1];

                    }
                    this.updateCount();
                    this.saveLabels();
                });

            },

            nextIndexUnlabelised() {
                let i = 1;
                while (i < this.nbImages && !isNaN(this.labels[this.images[(i + this.index) % this.nbImages].filename])) {
                    i++;
                }
                if (i === this.nbImages) {
                    this.modeIndiv = false;
                    this.index = 0;
                } else {
                    this.index = (i + this.index) % this.nbImages;
                }
            },

            enterPressed() {
                this.labels[this.image.filename] = isNaN(this.count) ? 0 : Number(this.count);
                this.saveLabels();
                this.rightPressed();
            },

            spacePressed() {
                if (this.count === undefined) {
                    this.count = 0;
                }
                this.count += 1;
            },

            leftPressed() {
                if (this.index > 0) {
                    this.index -= 1;
                }
            },

            rightPressed() {
                if (this.index < this.nbImages) {
                    this.index += 1;
                }
            },

            upPressed() {
                this.nextIndexUnlabelised();
            },

            downPressed() {
                delete this.labels[this.image.filename];
                this.count = undefined;
                this.saveLabels();
            },

            updateCount() {
                if (this.nbImages > 0) {
                    this.count = this.labels[this.images[this.index].filename];
                } else {
                    this.count = undefined;
                }
            },

            saveLabels() {
                localStorage.setItem('labels', JSON.stringify(this.labels));
            },

            clearLabels() {
                this.labels = {};
                this.saveLabels();
                this.labelsTable = [];
            },
        },

        computed: {
            showImageNumber: function () {
                return (this.nbImages !== 0) && this.modeIndiv;
            },

            nbImages: function () {
                return this.images.length;
            },

            image: function () {
                return this.images[this.index];
            },

            nbLabelised: function () {
                this.index;
                this.count;
                let c = 0;
                for (let image of this.images) {
                    if (!isNaN(this.labels[image.filename])) {
                        c++;
                    }
                }
                return c;
            },

            portrait: function () {
                return (this.$vuetify.breakpoint.width < this.$vuetify.breakpoint.height)
            },
        },

        watch: {
            index: function () {
                if (this.index === this.images.length) {
                    this.modeIndiv = false;
                    this.index = 0;
                }
                this.updateCount();
            },

            dialog: function () {
                let arr = [];
                for (let key in this.labels) {
                    arr.push({filename: key, label: this.labels[key]});
                }
                this.labelsTable = arr;
            },

        }
    };

</script>
