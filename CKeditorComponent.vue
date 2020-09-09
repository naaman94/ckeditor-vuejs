<template>
    <div>
        <div class="demo-update" :id="this.ids+'demo-update'">
            <!--    <CKeditorComponent v-model="editor" :max="50"/>-->

            <textarea name="content" :placeholder="placeholder||'ابدأ الكتابة'"
                      :id="ids+'-update__editor'">
            </textarea>
            <div class="demo-update__controls mt-1">
                <span class="demo-update__words" :id="this.ids+'demo-update__word'"/>
                <svg class="demo-update__chart" viewbox="0 0 40 40" width="40" height="40"
                     xmlns="http://www.w3.org/2000/svg">
                    <circle stroke="hsl(0, 0%, 93%)" stroke-width="3" fill="none" cx="20" cy="20" r="17"/>
                    <circle class="demo-update__chart__circle" :id="this.ids+'demo-update__chart__circle'"
                            stroke="hsl(202, 92%, 59%)" stroke-width="3"
                            stroke-dasharray="134,534" stroke-linecap="round" fill="none" cx="20" cy="20" r="17"/>
                    <text class="demo-update__chart__characters" :id="this.ids+'demo-update__chart__characters'" x="50%"
                          y="50%" dominant-baseline="central"
                          text-anchor="middle"/>
                </svg>
            </div>
        </div>
    </div>

</template>

<script>
    import Ckeditor from "./build/ckeditor";
    // var CK require()
    export default {
        props: ['value', 'max', 'placeholder', 'ids','initData'],
        methods: {},
        created: function () {

        },
        mounted() {
            const maxCharacters = this.max;
            const container = document.querySelector('#' + this.ids + 'demo-update');
            const progressCircle = document.querySelector('#' + this.ids + 'demo-update__chart__circle');
            const charactersBox = document.querySelector('#' + this.ids + 'demo-update__chart__characters');
            const wordsBox = document.querySelector('#' + this.ids + 'demo-update__word');
            const circleCircumference = Math.floor(2 * Math.PI * progressCircle.getAttribute('r'));

            ClassicEditor
                .create(document.querySelector('#' + this.ids + '-update__editor'), {
                    toolbar: {
                        items: [
                            'bold',
                            'italic',
                            'underline',
                            '|',
                            'fontSize',
                            'fontColor',
                            '|',

                            'bulletedList',
                            'numberedList',
                            '|',
                            'alignment',
                            'indent',
                            'outdent',
                            '|',
                            // 'mediaEmbed',
                            'link',
                            '|',
                            'MathType',
                            'ChemType',
                            'specialCharacters',
                            '|',
                            'undo',
                            'redo'
                        ]
                    },
                    language: 'ar',
                    licenseKey: '',


                    wordCount: {
                        onUpdate: stats => {
                            const charactersProgress = stats.characters / maxCharacters * circleCircumference;
                            const isLimitExceeded = stats.characters > maxCharacters;
                            const isCloseToLimit = !isLimitExceeded && stats.characters > maxCharacters * .8;
                            const circleDashArray = Math.min(charactersProgress, circleCircumference);

                            // Set the stroke of the circle to show how many characters were typed.
                            progressCircle.setAttribute('stroke-dasharray', `${circleDashArray},${circleCircumference}`);

                            // Display the number of characters in the progress chart. When the limit is exceeded,
                            // display how many characters should be removed.
                            if (isLimitExceeded) {
                                charactersBox.textContent = `-${stats.characters - maxCharacters}`;
                            } else {
                                charactersBox.textContent = maxCharacters - stats.characters;
                            }

                            wordsBox.textContent = `عدد الكلمات: ${stats.words}`;

                            // If the content length is close to the character limit, add a CSS class to warn the user.
                            container.classList.toggle('demo-update__limit-close', isCloseToLimit);

                            // If the character limit is exceeded, add a CSS class that makes the content's background red.
                            container.classList.toggle('demo-update__limit-exceeded', isLimitExceeded);

                            this.$emit('onChangeChar', stats)
                        }
                    }
                })
                .then(editor => {
                    editor.setData(this.initData || "");
                    this.$emit('input', editor)
                    // const wordCountPlugin = editor.plugins.get('WordCount').characters;
                })
                .catch(error => {
                    console.error('Oops, something gone wrong!');
                    console.error('Please, report the following error in the https://github.com/ckeditor/ckeditor5 with the build id and the error stack trace:');
                    console.warn('Build id: 40buqvp13p7y-uuhnvkxaieyq');
                    console.error(error);
                });

        }
    };
</script>
<style scoped>
    .demo-update {
        width: 100%;
        /*max-width: 500px;*/
    }

    .demo-update h3 {
        font-size: 18px;
        font-weight: bold;
        margin: 0 0 .5em;
        padding: 0;
    }

    .demo-update .ck.ck-editor__editable_inline {
        border: 1px solid hsla(0, 0%, 0%, 0.15);
        transition: background .5s ease-out;
        min-height: 6em;
        margin-bottom: 1em;
    }

    .demo-update__controls {
        display: flex;
        flex-direction: row;
        align-items: center;
    }

    .demo-update__chart {
        margin-right: 1em;
    }

    .demo-update__chart__circle {
        transform: rotate(-90deg);
        transform-origin: center;
    }

    .demo-update__chart__characters {
        font-size: 13px;
        font-weight: bold;
    }

    .demo-update__words {
        flex-grow: 1;
        opacity: .5;
    }

    .demo-update__limit-close .demo-update__chart__circle {
        stroke: hsl(30, 100%, 52%);
    }

    .demo-update__limit-exceeded .ck.ck-editor__editable_inline {
        background: hsl(0, 100%, 97%);
    }

    .demo-update__limit-exceeded .demo-update__chart__circle {
        stroke: hsl(0, 100%, 52%);
    }

    .demo-update__limit-exceeded .demo-update__chart__characters {
        fill: hsl(0, 100%, 52%);
    }
</style>
