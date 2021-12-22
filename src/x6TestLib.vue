<template>
  <div>
      <x6 ref="antvx6"></x6>
  </div>
</template>
<script>
const a = [
    {
        "shape": "edge",
        "attrs": {
            "line": {
                "stroke": "#7c68fc",
                "targetMarker": {
                    "name": "path",
                    "args": {
                        "size": "6"
                    }
                },
                "strokeDasharray": ""
            }
        },
        "id": "7b01a9a6-83b1-41fa-aba3-2bfdafc68220",
        "zIndex": -1,
        "source": {
            "cell": "b0a27800-e298-415c-803b-872cd63d85c4",
            "port": "group13"
        },
        "target": {
            "cell": "64c7c2ca-6ea8-4572-84aa-783064e1201f",
            "port": "group42"
        },
        "labels": [
            {
                "attrs": {
                    "label": {
                        "text": ""
                    }
                }
            }
        ]
    },
    {
        "position": {
            "x": 150,
            "y": 80
        },
        "size": {
            "width": 100,
            "height": 100
        },
        "view": "html-view",
        "shape": "html",
        "id": "b0a27800-e298-415c-803b-872cd63d85c4",
        "data": {
            "typeImgNo": "img1",
            "textContent": "基本事件",
            "defaultSpanStyle": ""
        },
        "ports": {
            "items": [
                {
                    "group": "group1",
                    "id": "group10",
                    "args": {
                        "x": "50%",
                        "y": 0
                    }
                },
                {
                    "group": "group1",
                    "id": "group11",
                    "args": {
                        "x": "50%",
                        "y": "100%"
                    }
                },
                {
                    "group": "group1",
                    "id": "group12",
                    "args": {
                        "x": 0,
                        "y": "50%"
                    }
                },
                {
                    "group": "group1",
                    "id": "group13",
                    "args": {
                        "x": "100%",
                        "y": "50%"
                    }
                }
            ],
            "groups": {
                "group1": {
                    "attrs": {
                        "circle": {
                            "r": 6,
                            "magnet": true,
                            "stroke": "#31d0c6",
                            "fill": "#fff",
                            "strokeWidth": 2
                        }
                    },
                    "position": {
                        "name": "absolute"
                    }
                }
            }
        },
        "zIndex": 1
    },
    {
        "position": {
            "x": 380,
            "y": 70
        },
        "size": {
            "width": 100,
            "height": 100
        },
        "view": "html-view",
        "shape": "html",
        "id": "64c7c2ca-6ea8-4572-84aa-783064e1201f",
        "data": {
            "typeImgNo": "img2",
            "textContent": "未探明事件",
            "defaultSpanStyle": ""
        },
        "ports": {
            "items": [
                {
                    "group": "group4",
                    "id": "group40",
                    "args": {
                        "x": "50%",
                        "y": "26%"
                    }
                },
                {
                    "group": "group4",
                    "id": "group41",
                    "args": {
                        "x": "50%",
                        "y": "74%"
                    }
                },
                {
                    "group": "group4",
                    "id": "group42",
                    "args": {
                        "x": "5%",
                        "y": "50%"
                    },
                    "connected": true
                },
                {
                    "group": "group4",
                    "id": "group43",
                    "args": {
                        "x": "95%",
                        "y": "50%"
                    }
                }
            ],
            "groups": {
                "group4": {
                    "attrs": {
                        "circle": {
                            "r": 6,
                            "magnet": true,
                            "stroke": "#31d0c6",
                            "fill": "#fff",
                            "strokeWidth": 2
                        }
                    },
                    "position": {
                        "name": "absolute"
                    }
                }
            }
        },
        "zIndex": 2
    }
]
import {x6} from "../lib/obstacle-wl.common.js"
  export default {
    name: "x6TestLib",
      data() {
        return {
        }
      },
      mounted(){
        this.$refs.antvx6.renderData(a)
      },
      components: {
          x6
      },
      methods: {

      }
  }
</script>
<style lang='less' scoped>
</style>