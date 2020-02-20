<template>
    <div class="geo_select">
        <div class="geo_select_input">
            <!-- General Input -->
            <input
                type="text"
                v-model="targetValue"
                class="geo_select_input_target"
                :id="id"
                :disabled="disable"
                @focus="focus = true"
                @blur="focus = false"
                @input="dataInitial = ''"
            >
            <!-- Transforming Label -->
            <label
                class="geo_select_input_label"
                :class="{ 'geo_select_input_label--active': targetValue || focus || dataInitial }"
                :for="id"
            >
                {{ label }}
            </label>
            <!-- Initial Value Label -->
            <label
                class="geo_select_input_initial"
                v-if="!targetValue"
                :class="{ 'geo_select_input_initial--focus': focus }"
                :for="id"
            >
                {{ dataInitial }}
            </label>
            <!-- Indicator -->
            <svg xmlns="http://www.w3.org/2000/svg" width="14" height="10" role="presentation" class="geo_select_input_indicator" :class="{ 'geo_select_input_indicator--active': focus }">
                <path d="M9.211364 7.59931l4.48338-4.867229c.407008-.441854.407008-1.158247 0-1.60046l-.73712-.80023c-.407008-.441854-1.066904-.441854-1.474243 0L7 5.198617 2.51662.33139c-.407008-.441853-1.066904-.441853-1.474243 0l-.737121.80023c-.407008.441854-.407008 1.158248 0 1.600461l4.48338 4.867228L7 10l2.211364-2.40069z"></path>
            </svg>
        </div>
        <!-- Options Container -->
        <div class="geo_select_container" v-if="options.length">
            <div
                class="geo_select_container_option"
                v-for="(option, index) in options"
                :key="index"
                @click="change(option.value, option.city)"
            >
                {{ option.value }}
            </div>
        </div>
    </div>
</template>

<script>
import { loadYmap } from 'vue-yandex-maps'

const settings = {
  apiKey: '03219f2d-b9b3-4850-98ea-9db0670d8a6b',
  lang: 'ru_RU',
  coordorder: 'latlong',
  version: '2.1'
}

export default {
    props: {
        id: String,
        label: String,
        initial: String
    },
    data() {
        return {
            ymapsMap: {},
            targetValue: '',
            options: [],
            promiseHub: [],
            focus: false,
            disable: true,
            dataInitial: '',
            bounds: {
                100000003:[ // Москва
                    [55.142627, 36.803259],
                    [56.021281, 37.967682]
                ],
                100000006:[ // Самара
                    [53.091785, 49.731327],
                    [53.550907, 50.390439]
                ],
                100000008:[ // Сочи
                    [43.384759, 39.149676],
                    [44.029925, 40.010388]
                ]
            }
        }
    },
    created() {
        this.dataInitial = this.initial
    },
    async mounted() {
        await loadYmap({ ...settings, debug: true})

        ymaps.ready(() => {
            this.disable = false
            this.ymapsMap = ymaps
        })
    },
    methods: {
        suggest(request) {
            const options = { boundedBy: this.bounds[100000003], results: 25 }

            this.ymapsMap.suggest(request, options)
                .then(items => {
                    let results = [],
                        promises = []

                    items.forEach(item => {
                        promises.push(this.ymapsMap.geocode(item.value)
                            .then(gc => {
                                if (!item.value.match(/.*подъезд.*/)) {
                                    let geoObject = gc.geoObjects.get(0)

                                    if (geoObject) {
                                        if (geoObject.getCountryCode() == "RU" && geoObject.getPremiseNumber() == null) {
                                            results.push({
                                                value: geoObject.getThoroughfare() || geoObject.getLocalities()[0],
                                                city: geoObject.getLocalities()[0] || geoObject.getAdministrativeAreas()[0]
                                            })
                                        }
                                    }
                                }
                            }))
                    })

                    Promise.all(promises).then(() => {
                        this.options = results
                    })
                })
        },
        change(value, city) {
            this.targetValue = ''
            this.dataInitial = value
            this.options = []
            localStorage.setItem('location.city', value)
            this.$emit('input', value)
        }
    },
    watch: {
        targetValue(val) {
            this.suggest(val)
        }
    }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Open+Sans&display=swap');

.geo_select {
    font-family: 'Open Sans', sans-serif;
    font-size: 14px;
    position: relative;

    &_input {
        position: relative;

        &_target {
            min-width: 360px;
            min-height: 48px;
            background-color: #fff;
            padding: 0 10px;
            border: 1px solid #cbcbcb;
            border-radius: 2px;
            cursor: pointer;
        }

        &_label {
            font-size: 14px;
            color: #bdbdbd;
            position: absolute;
            top: 50%;
            left: 10px;
            transform: translateY(-50%);
            transition: all .3s;
            cursor: pointer;

            &--active {
                display: block;
                background-color: #fff;
                font-size: 10px;
                padding: 0 3px;
                top: 0;
                left: 7px;
            }
        }

        &_initial {
            font-size: 14px;
            color: #bdbdbd;
            position: absolute;
            top: 50%;
            left: 10px;
            transform: translateY(-50%);
            transition: all .3s;
            cursor: pointer;

            &--focus {
                opacity: .5;
            }
        }

        &_indicator {
            position: absolute;
            top: 50%;
            right: 10px;
            transform-origin: 7px 2px;
            transform: translateY(-50%);
            transition: all .25s;
            
            path {
                fill: #bdbdbd;
            }

            &--active {
                transform: rotate(180deg);
            }
        }
    }

    &_container {
        min-width: 160px;
        max-height: 350px;
        width: calc(100% - 2px);
        display: flex;
        flex-direction: column;
        align-items: stretch;
        background-color: #fff;
        padding: 5px 0;
        box-shadow:0 3px 6px 0 rgba(0,0,0,.15);
        border:1px solid rgba(60,60,60,.26);
        border-top-style:none;
        border-radius:0 0 4px 4px;
        position: absolute;
        top: calc(100% - 1px);
        left: 0;
        overflow-y: auto;
        z-index: 1000;

        &_option {
            display:block;
            padding:3px 20px;
            clear:both;
            color:#333;
            line-height:1.42857143;
            white-space:nowrap;
            cursor: pointer;

            &:hover {
                background-color:#5897fb;
                color:#fff;
            }
        }
    }
}
</style>