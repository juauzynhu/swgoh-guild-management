<style scoped>
    .action-link {
        cursor: pointer;
    }
    .loader {
    position: relative;
    text-align: center;
    margin: 15px auto 35px auto;
    z-index: 9999;
    display: block;
    width: 80px;
    height: 80px;
    border: 10px solid rgba(0, 0, 0, .3);
    border-radius: 50%;
    border-top-color: #000;
    animation: spin 1s ease-in-out infinite;
    -webkit-animation: spin 1s ease-in-out infinite;
    }

    @keyframes spin {
    to {
        -webkit-transform: rotate(360deg);
    }
    }

    @-webkit-keyframes spin {
    to {
        -webkit-transform: rotate(360deg);
    }
    }


    /** MODAL STYLING **/

    .modal-content {
    border-radius: 0px;
    box-shadow: 0 0 20px 8px rgba(0, 0, 0, 0.7);
    }

    .modal-backdrop.show {
    opacity: 0.75;
    }

    .loader-txt p {
        font-size: 13px;
        color: #666;
    }
    .loader-txt small {
        font-size: 11.5px;
        color: #999;
        }

</style>

<template>
    <div>
        <div class="card mb-3">
            <div class="card-header">
                <div style="display: flex; justify-content: space-between; align-items: center;">
                    <span>
                        Memos
                    </span>
                    <div>
                    <a class="action-link" tabindex="-1" @click="getMemos">
                        Übersicht neu laden
                    </a>
                    <a class="action-link" tabindex="-1" @click="toggleTimer">
                        <span id="reload-label">(-)</span>
                    </a>
                    </div>

                    <a class="action-link" tabindex="-1" @click="showCreate">
                        Create New Memo
                    </a>
                </div>
            </div>

            <div class="card-body">
                <!-- Current Memos -->
                <p class="mb-0" v-if="memos.length === 0">
                    You have not created any memos.
                </p>

                <table class="table table-borderless mb-0" v-if="memos.length > 0">
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Titel</th>
                            <th>Auszug (Inhalt)</th>
                            <th>Erstellt / Geändert</th>
                            <th>Blockiert</th>
                            <th></th>
                            <th></th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr v-for="memo in memos" :key="memo.id">
                            <!-- ID -->
                            <td style="vertical-align: middle;">
                                {{ memo.id }}
                            </td>

                            <!-- Name -->
                            <td style="vertical-align: middle;">
                                {{ memo.title }}
                            </td>

                            <!-- Secret -->
                            <td style="vertical-align: middle;">
                                <code>{{ memo.body | truncate(25) }}</code>
                            </td>

                            <!-- Secret -->
                            <td style="vertical-align: middle;">
                                <span v-if="hasCreator(memo)">{{ memo.created_at }} ({{ memo.creator.name }})</span>
                                <span v-if="hasEditor(memo)"><br />{{ memo.updated_at }} ({{ memo.editor.name }})</span>
                            </td>

                            <!-- Secret -->
                            <td style="vertical-align: middle;">
                                <span v-if="hasLock(memo)">{{ memo.lock.created_at }} ({{ memo.lock.user.name }})<br />{{ memo.lock.updated_at }}</span>
                            </td>

                            <!-- Edit Button -->
                            <td style="vertical-align: middle;">
                                <a class="btn btn-success" @click="showEdit(memo)" aria-label="Edit">
                                    <i class="fa fa-pencil" aria-hidden="true"></i>
                                </a>
                                <!-- <a class="action-link" tabindex="-1" @click="showEdit(memo)">
                                    Edit
                                </a> -->
                            </td>

                            <!-- Position Button -->
                            <td style="vertical-align: middle;">
                                <a class="btn btn-secondary" @click="showPosition(memo)" aria-label="Position">
                                    <i class="fa fa-arrows" aria-hidden="true"></i>
                                </a>
                                <!-- <a class="action-link" @click="showPosition(memo)">
                                    Position
                                </a> -->
                            </td>

                            <!-- Delete Button -->
                            <td style="vertical-align: middle;">
                                <a class="btn btn-danger" @click="destroy(memo)" aria-label="Delete">
                                    <i class="fa fa-trash-o" aria-hidden="true"></i>
                                </a>
                                <!-- <a class="action-link text-danger" @click="destroy(memo)">
                                    Delete
                                </a> -->
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <!-- Create Memo Modal -->
        <!-- tabindex="-1" not allowed, breaks fokus for editors -->
        <div class="modal fade" id="modal-create-memo" role="dialog">
            <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <div class="modal-header">
                        <h4 class="modal-title">
                            Create Memo
                        </h4>

                        <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
                    </div>

                    <div class="modal-body">
                        <!-- Form Errors -->
                        <div class="alert alert-danger" v-if="createForm.errors.length > 0">
                            <p class="mb-0"><strong>Whoops!</strong> Something went wrong!</p>
                            <br>
                            <ul>
                                <li v-for="(error, index) in createForm.errors" :key="`error-${index}`">
                                    {{ error }}
                                </li>
                            </ul>
                        </div>


                        <div>
                            <div v-if="signedIn">
                                <!-- Create Memo Form -->
                                <form role="form">
                                    <!-- Name -->
                                    <div class="form-group">
                                        <!-- <label class="col-md-3 col-form-label">Title</label> -->

                                        <!-- <div class="col-md-12"> -->
                                            <input id="create-memo-title" type="text" class="form-control" name="title" v-model="createForm.title">

                                            <span class="form-text text-muted">Section title.</span>
                                        <!-- </div> -->
                                    </div>
                                    <div class="form-group">
<ckeditor name="body" id="create-memo-body" v-model="createForm.body" placeholder="Bitte Text eingeben..." :shouldClear="createForm.clearToggle"></ckeditor>
                                    </div>

                                </form>
                            </div>
                            <p class="text-center" v-else>
                                Please <a href="/login">sign in</a> to participate in this
                                discussion.
                            </p>
                        </div>
                    </div>

                    <!-- Modal Actions -->
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>

                        <button type="button" class="btn btn-primary" @click="store">
                            Create
                        </button>
                        <!-- <button type="submit"
                                class="btn btn-secondary"
                                @click="addReply">Post</button> -->
                    </div>

                </div>
            </div>
        </div>

        <!-- Edit Memo Modal -->
        <!-- tabindex="-1" not allowed, breaks fokus for editors -->
        <div class="modal fade" id="modal-edit-memo" role="dialog">
            <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <div class="modal-header">
                        <h4 class="modal-title">
                            Edit Memo
                        </h4>

                        <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
                    </div>

                    <div class="modal-body">
                        <!-- Form Errors -->
                        <div class="alert alert-danger" v-if="editForm.errors.length > 0">
                            <p class="mb-0"><strong>Whoops!</strong> Something went wrong!</p>
                            <br>
                            <ul>
                                <li v-for="(error, index) in editForm.errors" :key="`error-${index}`">
                                    {{ error }}
                                </li>
                            </ul>
                        </div>

                        <!-- Edit Memo Form -->
                        <form role="form">
                            <!-- Name -->
                            <div class="form-group xxxrow">
                                <!-- <label class="col-md-3 col-form-label">Title</label> -->

                                <!-- <div class="col-md-9"> -->
                                    <input id="edit-memo-title" type="text" class="form-control" v-model="editForm.title">

                                    <span class="form-text text-muted">
                                        Titel des Abschnitts.
                                    </span>
                                <!-- </div> -->
                            </div>

                            <!-- Redirect URL -->
                            <div class="form-group xxxrow">
                                <!-- <label class="col-md-3 col-form-label">Text</label> -->

                                <!-- <div class="col-md-9"> -->
<ckeditor name="body" id="edit-memo-body" v-model="editForm.body" placeholder="Bitte Text eingeben..." :should-clear="editForm.clearToggle"></ckeditor>
                                    <span class="form-text text-muted">
                                        Inhalt / Text des Abschnitts.
                                    </span>
                                <!-- </div> -->
                            </div>
                        </form>
                    </div>

                    <!-- Modal Actions -->
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>

                        <button type="button" class="btn btn-primary" @click="update">
                            Save Changes
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Position Memo Modal -->
        <!-- tabindex="-1" not allowed, breaks fokus for editors -->
        <div class="modal fade" id="modal-position-memo" role="dialog">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h4 class="modal-title">
                            Change Position ({{ positionForm.title }})
                        </h4>

                        <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
                    </div>

                    <div class="modal-body">
                        <!-- Form Errors -->
                        <div class="alert alert-danger" v-if="positionForm.errors.length > 0">
                            <p class="mb-0"><strong>Whoops!</strong> Something went wrong!</p>
                            <br>
                            <ul>
                                <li v-for="(error, index) in positionForm.errors" :key="`error-${index}`">
                                    {{ error }}
                                </li>
                            </ul>
                        </div>

                        <!-- Position Memo Form -->
                        <form role="form">
                            <!-- Name -->
                            <div class="form-group row">
                                <label class="col-md-3 col-form-label">Page</label>

                                <div class="col-md-9">
                                    <span class="form-text">Aktualität der Listen: {{ pages_time | formatDate }} <span class="btn btn-link" @click="getPages">(refresh)</span></span>
                                    
                                    <select v-model="positionForm.page_id">
                                    <option v-for="page in pages" v-bind:value="page.id" :key="page.id">
                                        {{ page.title }}
                                    </option>
                                    </select>
                                    <span>Selected: {{ positionForm.page_id }}</span>
                                    
                                    <span class="form-text text-muted">
                                        Seite, auf der der Abschnitt angezeigt wird.
                                    </span>
                                </div>
                            </div>

                            <!-- Redirect URL -->
                            <div class="form-group row">
                                <label class="col-md-3 col-form-label">Nachfolger</label>

                                <div class="col-md-9">
                                    <select v-model="positionForm.before_id">
                                    <option value="">
                                        - ans Ende -
                                    </option>
                                    <option v-for="page_memo in page_memos" v-bind:value="page_memo.id" :key="page_memo.id">
                                        {{ page_memo.title }}
                                    </option>
                                    </select>
                                    <span>Selected: {{ positionForm.before_id }}</span>

                                    <span class="form-text text-muted">
                                        Abschnitt, VOR dem dieser Abschnitt angezeigt/eingeordnet werden soll.
                                    </span>
                                </div>
                            </div>
                        </form>
                    </div>

                    <!-- Modal Actions -->
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>

                        <button type="button" class="btn btn-primary" @click="relocate">
                            Save Changes
                        </button>
                    </div>
                </div>
            </div>
        </div>

    
        <!-- Modal -->
        <div class="modal fade" id="loadMe" tabindex="-1" role="dialog" aria-labelledby="loadMeLabel">
        <div class="modal-dialog modal-sm" role="document">
            <div class="modal-content">
            <div class="modal-body text-center">
                <div class="loader"></div>
                <div class="loader-txt">
                <i class="fa fa-rebel fa-3x" aria-hidden="true"></i>
                <p>Patience you must have, my young Padawan.</p>
                <p><small>Time will tell when loading is finished... #yoda</small></p>
                <i class="fa fa-empire fa-3x" aria-hidden="true"></i>
                </div>
            </div>
            </div>
        </div>
        </div>

    <!-- end surrounding div -->
    </div>
</template>

<script>
// https://forum.vuejs.org/t/v-for-with-simple-arrays-what-key-to-use/13692/2
// https://github.com/vuejs/eslint-plugin-vue/blob/master/docs/rules/require-v-for-key.md
// pref: v-for="(error, index) in createForm.errors" :key="`error-${index}`"
// var2: v-for="(item, index) in items" :key="index"
    import ckeditor from '../components/ckeditor.vue';
    import 'jquery.caret';
    import 'at.js';
    import moment,{ now } from 'moment';
    export default {
        props: ['name', 'value', 'placeholder'],
        components: { ckeditor },
        /*
         * The component's data.
         */
        data() {
            return {
                memos: [],
                page_memos: [],
                pages: [],
                pages_time: 0,

                createForm: {
                    errors: [],
                    clearToggle: false,
                    title: '',
                    body: ''
                },

                editForm: {
                    errors: [],
                    clearToggle: false,
                    id: 0,
                    title: '',
                    body: ''
                },

                positionForm: {
                    errors: [],
                    clearToggle: false,
                    id: 0,
                    title: '',
                    page_id: 0,
                    before_id: 0
                },

                timer: {
                    value: 0,
                    count: 0,
                    enabled: true,
                    requestrunning: false,
                    interval: 60,
                    refresh: 1000,
                    limit: 20
                }
            };
        },

        /**
         * Prepare the component (Vue 2.x).
         */
        mounted() {
            this.prepareComponent();

            $('#create-memo-body').atwho({
                at: "@",
                delay: 750,
                callbacks: {
                    remoteFilter: function(query, callback) {
                        $.getJSON("/api/users", {name: query}, function(usernames) {
                            callback(usernames)
                        });
                    }
                }
            });
            $('#edit-memo-body').atwho({
                at: "@",
                delay: 750,
                callbacks: {
                    remoteFilter: function(query, callback) {
                        $.getJSON("/api/users", {name: query}, function(usernames) {
                            callback(usernames)
                        });
                    }
                }
            });
        },

        computed: {
            ago() {
                return moment(this.pages_time).fromNow() + '...';
            }
        },

        filters: {
            formatDate: function(value) {
                if (value) {
                    return moment(value).format('MM.DD.YYYY H:m:s')
                }
            }
        },

        watch: {
            'positionForm.page_id': function() {
                this.getOtherMemos(this.positionForm.page_id);
                this.positionForm.before_id = '';
            }
        },

        methods: {
            clearCreate() {
                this.createForm.errors = [];
                this.createForm.title = '';
                // this.createForm.body = ''; //crashes on ckeditor
                this.createForm.clearToggle = ! this.createForm.clearToggle;
            },
            clearEdit() {
                this.editForm.errors = [];
                this.editForm.id = 0;
                this.editForm.title = '';
                // this.editForm.body = ''; //crashes on ckeditor
                this.editForm.clearToggle = ! this.editForm.clearToggle;
            },
            clearPosition() {
                this.positionForm.errors = [];
                this.positionForm.id = 0;
                this.positionForm.title = '';
                this.positionForm.page_id = 0;
                this.positionForm.before_id = 0;
            },
            clearPages() {
                //clear list of pages
                this.pages = [];
                this.pages_time = 0;

            },
            hasCreator(memo) {
                // return !!this.$slots.default[0].text.length;
                return typeof memo.creator !== 'undefined' && memo.creator !== null;
            },
            hasEditor(memo) {
                return typeof memo.editor !== 'undefined' && memo.editor !== null;
            },
            hasLock(memo) {
                return typeof memo.lock !== 'undefined' && memo.lock !== null;
            },
            async checkTimer() {
                if (this.timer.count > this.timer.limit || ! this.timer.enabled) {
                    document.getElementById('reload-label').innerHTML = "(auto reload off)";
                    return;
                }
                var left = this.timer.value;
                if (left == 1) {
                    document.getElementById('reload-label').innerHTML = "...";
                    await this.getMemos(false);
                    this.timer.count = this.timer.count + 1;
                    this.resetTimer();
                } else if (left < 1) {
                    // should never happen
                    // if it does, sth is wrong so do nothing and get out
                    return;
                } else {
                    var h = Math.floor(left / 60 / 60);
                    var m = Math.floor( (left % 60) / 60);
                    var s = (left % 120);
                    document.getElementById('reload-label').innerHTML = h + ":" + m + ":" + s;
                    var t = setTimeout(this.checkTimer, this.timer.refresh);
                    this.timer.value = this.timer.value - 1;
                }
            },
            toggleTimer() {
                this.timer.enabled = ! this.timer.enabled;
                if (this.timer.enabled) {
                    this.resetTimer(true);
                }
            },
            resetTimer(resetcounter = false) {
                if (resetcounter) {
                    this.timer.count = 0;
                }
                this.timer.value = this.timer.interval;
                this.checkTimer();
            },
            /**
             * Prepare the component.
             */
            prepareComponent() {

                $('#loadMe').on('shown.bs.modal', () => {
                    if (! this.timer.requestrunning) {
                        // Request ended faster than modal needed to show
                        // That means hide was possibly already called and got lost
                        console.log('Info', 'request faster than modal');
                        $("#loadMe").modal("hide");
                    }
                });

                $('#modal-create-memo').on('shown.bs.modal', () => {
                    $('#create-memo-title').focus();
                });

                $('#modal-edit-memo').on('shown.bs.modal', () => {
                    $('#edit-memo-title').focus();
                });

                $('#modal-create-memo').on('hidden.bs.modal', () => {
                    this.clearCreate();
                });
                // $('#modal-edit-memo').on('hide.bs.modal', () => {
                //     this.releaseLock(this.editForm.id);
                // });
                $('#modal-edit-memo').on('hidden.bs.modal', () => {
                    this.clearEdit();
                });
                $('#modal-position-memo').on('hidden.bs.modal', () => {
                    this.clearPosition();
                });

                // console.log(location.pathname); // w/o prot, w/o domain
                // console.log(location.origin); // prot + domain
                // console.log(location.href); // complete url
                this.getMemos();

                // this.getPages();

                this.resetTimer();

            },

            includeCkeditor() {
                // if (document.getElementById('my-ckeditor-include')) return; // was already loaded
                // var scriptTag = document.createElement("script");
                // scriptTag.src = "//cdn.ckeditor.com/4.10.1/standard/ckeditor.js";
                // scriptTag.id = "my-ckeditor-include";
                // document.getElementsByTagName('head')[0].appendChild(scriptTag);
            },

            /**
             * Get all of the memos for the user.
             */
            async getMemos(spin = true) {
                if (this.timer.requestrunning) {
                    return;
                }
                this.timer.requestrunning = true;
                if (spin) {
                    $("#loadMe").modal({
                    backdrop: "static", //remove ability to close modal with click
                    keyboard: false, //remove option to close with keyboard
                    show: true //Display loader!
                    });
                }
                await axios.get(location.pathname + '/memos')
                    .then(response => {
                        this.memos = response.data;
                        this.timer.value = this.timer.interval;
                        this.timer.requestrunning = false;
                        $("#loadMe").modal("hide");
                    })
                    .catch(error => {
                        this.timer.value = this.timer.interval;
                        this.timer.enabled = false;
                        this.timer.requestrunning = false;
                        $("#loadMe").modal("hide");
                        console.log('Error', error.message);
                        flash('Abruf der Inhalte fehlgeschlagen', 'danger');
                    });
            },

            /**
             * Get all of the memos for the user.
             */
            getOtherMemos(id) {
                if (! id || ! this.pages) {
                    return;
                }
                this.pages.forEach(page => {
                    if (page.id == id) {
                        this.page_memos = page.memos;
                        return;
                    }
                });
            },

            /**
             * Get all of the memos for the user.
             */
            getPages() {
                axios.get(location.pathname + '/pages')
                        .then(response => {
                            this.pages = response.data;
                            this.pages_time = moment.now();
                        });
            },

            /**
             * Get lock on memo for editing.
             */
            async getLock(id) {
                let lock = await axios.post('memos/' + id + '/lock', {}, {
                        headers: {
                            'Accept': 'application/json',
                            'Content-Type': 'application/json',
                        }
                    })
                    .then(response => {
                        flash('Sperre für Bearbeitung eingerichtet.', 'success');
                        return response.data;
                    })
                    .catch(error => {
                        if (error.response) {
                            console.log(error.response.data);
                            return false;
                        } else if (error.request) {
                            // The request was made but no response was received
                            console.log(error.request);
                        } else {
                            // Something happened in setting up the request that triggered an Error
                            console.log('Error', error.message);
                        }
                        flash('Bearbeitungssperre für Textelement fehlgeschlagen.', 'danger');
                        return false;
                    });
                return lock;
            },

            /**
             * Get lock on memo for editing.
             */
            releaseLock(id) {
                axios.delete('memos/' + id + '/lock', {}, {
                        headers: {
                            'Accept': 'application/json',
                            'Content-Type': 'application/json',
                        }
                    })
                    .then(response => {
                        flash(response.data, 'success');
                        return true;
                    })
                    .catch(error => {
                        if (error.response) {
                            console.log(error.response.data);
                        } else if (error.request) {
                            // The request was made but no response was received
                            console.log(error.request);
                        } else {
                            // Something happened in setting up the request that triggered an Error
                            console.log('Error', error.message);
                        }
                        flash('Die Bearbeitungssperre konnte nicht aufgehoben werden.', 'danger');
                    });
            },

            /**
             * Show the form for creating new memos.
             */
            showCreate() {
                $('#modal-create-memo').modal({
                    backdrop: "static", //remove ability to close modal with click
                    keyboard: false, //remove option to close with keyboard
                    show: true //Display loader!
                    });
            },

            /**
             * Create a new memo for the user.
             */
            store() {
                let data = new FormData();

                data.append('title', this.createForm.title);
                data.append('body', this.createForm.body);
                this.persistMemo(
                    'post', 'memos',
                    data, '#modal-create-memo', this.createForm
                );
            },

            /**
             * Edit the given memo.
             */
            async showEdit(memo) {
                let lock = await this.getLock(memo.id);

                if (!lock || typeof lock.user === 'undefined' ) {
                    flash('Eintrag wird bereits bearbeitet.', 'danger');
                    return false;
                }
                this.editForm.id = memo.id;
                this.editForm.title = memo.title;
                this.editForm.body = memo.body;

                $('#modal-edit-memo').modal({
                    backdrop: "static", //remove ability to close modal with click
                    keyboard: false, //remove option to close with keyboard
                    show: true //Display loader!
                    });
            },

            /**
             * Edit the given memo.
             */
            showPosition(memo) {
                this.positionForm.id = memo.id;
                this.positionForm.page_id = memo.page_id;
                this.positionForm.title = memo.title;
                // this.getOtherMemos(memo.page_id); //watch
                // if (! this.pages || this.pages.length == 0) {
                    this.pages = [];
                    this.getPages();
                // }

                $('#modal-position-memo').modal({
                    backdrop: "static", //remove ability to close modal with click
                    keyboard: false, //remove option to close with keyboard
                    show: true //Display loader!
                    });
            },

            /**
             * Update the memo being edited.
             */
            update() {
                let data = new FormData();

                data.append('title', this.editForm.title);
                data.append('body', this.editForm.body);
                this.persistMemo(
                    'put', 'memos/' + this.editForm.id,
                    data, '#modal-edit-memo', this.editForm
                );
            },

            /**
             * Update the memo being edited.
             */
            relocate() {
                let data = new FormData();

                data.append('page_id', this.positionForm.page_id);
                data.append('before_id', this.positionForm.before_id);
                this.persistMemo(
                    'put', 'memos/' + this.positionForm.id + '/relocate',
                    data, '#modal-position-memo', this.positionForm
                );
            },

            /**
             * Persist the memo to storage using the given form.
             */
            persistMemo(method, uri, data, modal, form) {
                // fix PUT for Laravel using _method field
                // https://github.com/laravel/framework/issues/13457
                // alternatively: use 'Content-Type': 'application/x-www-form-urlencoded'
                if (method == 'put') {
                    data.append('_method', 'PUT');
                    method = 'post';
                }
                axios({
                    method: method,
                    url: uri,
                    data: data,
                    headers: {
                        'Accept': 'application/json',
                        // 'Content-Type': 'application/x-www-form-urlencoded',
                        'Content-Type': 'multipart/form-data',
                        // 'Content-Type': 'application/json',
                        }
                })
                .then(response => {
                    $(modal).modal('hide');
                    switch (modal) {
                        case '#modal-create-memo':
                            this.clearPages();
                            this.getMemos();
                            flash('Your memo has been posted.', 'success');
                            break;
                    
                        case '#modal-edit-memo':
                            this.releaseLock(this.editForm.id);
                            this.clearPages();
                            this.getMemos();
                            flash('Your memo has been updated.', 'success');
                            break;
                    
                        case '#modal-position-memo':
                            this.clearPages();
                            this.getMemos();
                            flash('Your memo has been relocated.', 'success');
                            break;
                    
                        default:
                            flash('Something unknown was done successfully... ugh!?', 'success');
                            break;
                    }
                })
                .catch(error => {
                    if (error.response) {
                        form.errors = [];
                        // The request was made and the server responded with a status code
                        // that falls out of the range of 2xx
                        console.log(error.response.data);
                        console.log(error.response.status);
                        if (error.response.data.message) {
                            form.errors = [error.response.data.message];
                        } else {
                            form.errors = ['unknown error'];
                        }
                        flash(error.message, 'danger');
                    } else if (error.request) {
                        // The request was made but no response was received
                        // `error.request` is an instance of XMLHttpRequest in the browser and an instance of
                        // http.ClientRequest in node.js
                        form.errors = [error.message];
                        flash(error.message, 'danger');
                        console.log(error.request);
                    } else {
                        // Something happened in setting up the request that triggered an Error
                        console.log('Error', error.message);
                        form.errors = [error.message];
                        flash(error.message, 'danger');
                    }
                });
            },

            /**
             * Destroy the given memo.
             */
            destroy(memo) {
                axios.delete('memos/' + memo.id, {}, {
                        headers: {
                            'Accept': 'application/json',
                            // 'Content-Type': 'application/x-www-form-urlencoded',
                            // 'Content-Type': 'multipart/form-data',
                            'Content-Type': 'application/json',
                        }
                    }
                )
                .then(response => {
                    // this.getMemos();
                    let i = this.memos.map(item => item.id).indexOf(memo.id) // find index of your object
                    this.memos.splice(i, 1) // remove it from array
                    flash('Memo erfolgreich gelöscht.', 'success');
                })
                .catch(error => {
                    if (error.response) {
                        console.log(error.response.data);
                    } else if (error.request) {
                        console.log(error.request);
                    } else {
                        console.log('Error', error.message);
                    }
                    flash('Es ist ein Fehler beim Löschen aufgetreten.', 'danger');
                });
            }
        }
    }
</script>
