<template>
    <div ref="calendar" id="calendar">
        <grid-loader :loading="loading" :color="color" :size="size" class="loader"></grid-loader>
    </div>
</template>

<script>
    import defaultsDeep from 'lodash.defaultsdeep'
    import 'fullcalendar'
    import $ from 'jquery'
    import GridLoader from 'vue-spinner/src/GridLoader.vue'

    export default {
        components: {
            GridLoader
        },

        data() {
            return {
                loading: false,
                color: '#3AB982',
                size: "20px"
            }
        },
        props: {
            events: {
                default() {
                    return []
                },
            },

            eventSources: {
                default() {
                    return []
                },
            },

            editable: {
                default() {
                    return true
                },
            },

            selectable: {
                default() {
                    return true
                },
            },

            selectHelper: {
                default() {
                    return true
                },
            },

            header: {
                default() {
                    return {
                        left:   'prev,next today',
                        center: 'title',
                        right:  'month,agendaWeek,agendaDay'
                    }
                },
            },

            defaultView: {
                default() {
                    return 'month'
                },
            },

            sync: {
                default() {
                    return false
                }
            },

            config: {
                type: Object,
                default() {
                    return {}
                },
            },
            eventLimitClick: {
                default() {
                    return 'popover'
                },
            },
            eventLimit: {
               default() {
                    return false;
                }, 
            },
            eventLimitText: {
                default() {
                    return false;
                }
            },
            timezone: {
                default() {
                    return false;
                }
            },
            views: {
                default(){
                    return false;
                }
            }
        },

        computed: {
            defaultConfig() {
                const self = this
                return {
                    header: this.header,
                    defaultView: this.defaultView,
                    editable: this.editable,
                    selectable: this.selectable,
                    selectHelper: this.selectHelper,
                    aspectRatio: 2,
                    // timeFormat: 'HH:mm',
                    events: this.events,
                    eventSources: this.eventSources,
                    eventLimit: this.eventLimit,
                    eventLimitText: this.eventLimitText,
                    eventLimitClick: this.eventLimitClick,
                    views: this.views,
                    timezone: this.timezone,
                    displayEventEnd: true,
                    // nextDayThreshold:"00:00:00",
                    loading: function( isLoading, view ) {
                        // console.log(isLoading);
                        if(isLoading) {// isLoading gives boolean value
                            self.loading = true;
                        } else {
                            self.loading = false;
                        }
                    },
                    height: "auto",

                    eventRender(...args) {
                        if (this.sync) {
                            self.events = cal.fullCalendar('clientEvents')
                        }
                        self.$emit('event-render', ...args)
                    },

                    eventDestroy(event) {
                        if (this.sync) {
                            self.events = cal.fullCalendar('clientEvents')
                        }
                    },

                    eventClick(...args) {
                        self.$emit('event-selected', ...args)
                    },

                    eventDrop(...args) {
                        self.$emit('event-drop', ...args)
                    },

                    eventResize(...args) {
                        self.$emit('event-resize', ...args)
                    },

                    dayClick(...args){
                        self.$emit('day-click', ...args)
                    },
                    
                    select(start, end, jsEvent, view, resource) {
                        self.$emit('event-created', {
                            start,
                            end,
                            allDay: !start.hasTime() && !end.hasTime(),
                            view,
                            resource
                        })
                    }
                }
            },
        },

        mounted() {
            const cal = $(this.$el),
                self = this

            this.$on('remove-event', (event) => {
                if(event && event.hasOwnProperty('id')){
                    $(this.$el).fullCalendar('removeEvents', event.id);
                }else{
                    $(this.$el).fullCalendar('removeEvents', event);
                }
            })

            this.$on('rerender-events', () => {
                $(this.$el).fullCalendar('rerenderEvents')
            })

            this.$on('refetch-events', () => {
                $(this.$el).fullCalendar('refetchEvents')
            })

            this.$on('render-event', (event) => {
                $(this.$el).fullCalendar('renderEvent', event)
            })

            this.$on('reload-events', () => {
                $(this.$el).fullCalendar('removeEvents')
                $(this.$el).fullCalendar('addEventSource', this.events)
            })

            this.$on('rebuild-sources', () => {
                $(this.$el).fullCalendar('removeEventSources')
                this.eventSources.map(event => {
                    $(this.$el).fullCalendar('addEventSource', event)
                })
            })

            this.$on('add-event-sources', (events)=> {                
                $(this.$el).fullCalendar('addEventSource', events)
            })

            cal.fullCalendar(defaultsDeep(this.config, this.defaultConfig))
        },

        methods: {
            fireMethod(...options) {
                return $(this.$el).fullCalendar(...options)
            },
        },

        watch: {
            events: {
                deep: true,
                handler(val) {
                    $(this.$el).fullCalendar('removeEvents')
                    $(this.$el).fullCalendar('addEventSource', this.events)
                },
            },
            eventSources: {
                deep: true,
                handler(val) {
                    this.$emit('rebuild-sources')
                },
            },
        },

        beforeDestroy() {
            this.$off('remove-event')
            this.$off('rerender-events')
            this.$off('refetch-events')
            this.$off('render-event')
            this.$off('reload-events')
            this.$off('rebuild-sources')
        },
    }
</script>

<style lang="scss" scoped>
    #calendar {
        position: relative;
    }

    .loader {
        position: absolute;
        top: 50%;
        left: 50%;
        z-index: 1000;
    }
</style>

