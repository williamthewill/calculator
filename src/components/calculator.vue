<template>
	<div class="container">
		<!-- <div class="calculator">
			<div class="display">{{display || '0'}}</div>
			<div @click="clear" class="key clear">AC</div>
			<div @click="signal" class="key signal">+/-</div>
			<div @click="pressKey" class="key percentage">%</div>
			<div @click="pressKey" class="key operator division">÷</div>
			<div @click="pressKey" class="key number">7</div>
			<div @click="pressKey" class="key number">8</div>
			<div @click="pressKey" class="key number">9</div>
			<div @click="pressKey" class="key operator times">x</div>
			<div @click="pressKey" class="key number">4</div>
			<div @click="pressKey" class="key number">5</div>
			<div @click="pressKey" class="key number">6</div>
			<div @click="pressKey" class="key operator minus">-</div>
			<div @click="pressKey" class="key number">1</div>
			<div @click="pressKey" class="key number">2</div>
			<div @click="pressKey" class="key number">3</div>
			<div @click="pressKey" class="key operator sum">+</div>
			<div @click="pressKey" class="key number zero">0</div>
			<div @click="pressKey" class="key dot">.</div>
			<div @click="pressEqual" class="key equal">=</div>
		</div> -->
        <select @change="statusColumn()" v-model="dontShow">
            <option v-for="header in headers" :value="header">{{header.value}}</option>
        </select>
        <div>
            {{dontShow}}
        </div>
		<div class="grid" v-bind:style="{gridTemplateColumns: frColumns}">
			<div class="col" :class="'col-'+index" @click="sortBy(header)" draggable @dragstart="drag" @drop="drop" @dragover="dragover" v-for="(header,index,key) in headers">
                <div class="row header header-1">{{header}}<span @mousedown.prevent="resizeColumn"></span></div>
				<div class="row" v-for="item in filteredData">{{item[header]}}</div>
            </div>
		</div>
	</div>
</template>

<script lang="ts">
import Vue from 'vue';
import { setFlagsFromString } from 'v8';
import { setTimeout } from 'timers';
import { join } from 'path';

export default Vue.extend({
    name: 'calculator',
    props: {
        msg: String
    },
    data() {
        return {
            display: '',
            numberOne: '',
            operator: '',
            numberTwo: '',
            columnDrag: '',
            dontShow: '',
            frColumns: '',
            sortKey: '',
            sortOrders: new Object,
            filteredData: new Array,
            headers: [ 'horario', 'conta', 'url', 'localidade', 'data'
                // {value: 'horario', show: true},
                // {value: 'conta', show: true},
                // {value: 'url', show: true},
                // {value: 'localidade', show: true},
                // {value: 'data', show: true}
            ],
            fakeNews: [
                {
                    horario: 'horarioo',
                    conta: 'contaa',
                    url: 'urll',
                    localidade: 'localidadee',
                    data: 'data'
                },
                {
                    horario: 'horarioo1',
                    conta: 'contaa1',
                    url: 'urll1',
                    data: 'data1'
                },
                {
                    horario: 'horarioo',
                    conta: 'bontaa',
                    url: 'urll',
                    localidade: 'localidadee',
                    data: 'data'
                }
                                
            ]
        };
    },
    created: function() {
        this.headers.forEach(function (key) {
            this.sortOrders[key] = 0;
        }.bind(this));
        this.filteredData = this.fakeNews;
        window.addEventListener( 'load', function(event) {
            this.frColumns = this.templateColumns(this.headers);
        }.bind(this));
    },
    methods: {
        clear() {
            this.display = '';
            this.numberOne = '';
            this.numberTwo = '';
            this.operator = '';
        },
        pressKey(event: Event) {
            if (!event || !event.target) return;

            let domElement: HTMLDivElement = event.target as HTMLDivElement;

            if (
                domElement.innerHTML != '0' &&
                domElement.innerHTML != '.' &&
                !parseInt(domElement.innerHTML)
            ) {
                switch (domElement.innerHTML) {
                    case '÷':
                        this.operator = `/`;
                        break;
                    case 'x':
                        this.operator = `*`;
                        break;
                    default:
                        this.operator = domElement.innerHTML;
                        break;
                }
                return;
            }

            if (this.operator) {
                this.numberTwo = `${this.numberTwo}${domElement.innerHTML}`;
                this.display = this.numberTwo;
                return;
            }

            this.numberOne = `${this.numberOne}${domElement.innerHTML}`;
            this.display = this.numberOne;

        },
        sortBy(key) {
            this.sortKey = key
            if(this.sortOrders[key] === 0){
                this.sortOrders[key] = 1;
            } else if(this.sortOrders[key] === 1){
                this.sortOrders[key] = -1;
            } else {
                this.sortOrders[key] = 0;
            }
            this.filteredData = this.filter();
        },
        filter() {
            if(this.sortOrders[this.sortKey] === 0) {
                return this.fakeNews;
            }
            var order = this.sortOrders[this.sortKey] || 1;
            var data = this.fakeNews;
            if (this.sortKey) {
                data = data.slice().sort(function (a, b) {
                    a = a[this.sortKey];
                    b = b[this.sortKey];
                    return (a === b ? 0 : a > b ? 1 : -1) * order;
                }.bind(this))
            }
            return data
        },
        statusColumn() {
            this.dontShow.show = !this.dontShow.show;
            this.frColumns = this.templateColumns(Object.keys(this.headers.filter(function(obj) { return obj.show === true; })).length);
        },
        signal() {
            this.display =
                this.display.charAt(0) === '-' ? this.display.slice(1) : `-${this.display}`;
        },
        pressEqual() {
            this.display = eval(`${this.numberOne}${this.operator}${this.numberTwo}`);
        },
        hover(event: Event) {
            let elementHover = (document.onmouseover = domElement => domElement);
        },
        templateColumns(num) {            
            let templateColumns = '';
            for (let i = 0; i < num.length; i++){
                templateColumns += '1fr ';
            }
            return templateColumns;
        },
        drop(event: Event) {
            let columnDrop = event.srcElement.parentElement;
            let grid = columnDrop.parentElement;
            let nodes = Array.prototype.slice.call(grid.children);
            let indexToDrop = nodes.indexOf(columnDrop);
            let indexDrag = nodes.indexOf(this.columnDrag);
            let gridTemplateColumns = grid.style.gridTemplateColumns;
            console.log(gridTemplateColumns);
            gridTemplateColumns = gridTemplateColumns.split(' ');
            [gridTemplateColumns[indexDrag], gridTemplateColumns[indexToDrop]] = [
                gridTemplateColumns[indexToDrop],
                gridTemplateColumns[indexDrag]
            ];
            console.log(gridTemplateColumns);
            grid.style.gridTemplateColumns = gridTemplateColumns.join(' ');
            if (indexToDrop > indexDrag) {
                grid.insertBefore(this.columnDrag, grid.childNodes[indexToDrop + 1]);
            } else {
                grid.insertBefore(this.columnDrag, grid.childNodes[indexToDrop]);
            }
        },
        drag(event: Event) {
            this.columnDrag = event.srcElement;
        },
        dragover(event: Event) {
            event.preventDefault();
        },
        resizeColumn(event: Event) {
            event.preventDefault();
            let mouseMove = function(e) {
                let elementResizible = event.srcElement.parentElement.parentElement;
                let grid = document.querySelector('.grid');

                let elem = document.querySelector(
                    '.' + elementResizible.className.replace(/ /g, '.')
                );
                let nodes = Array.prototype.slice.call(grid.children);
                let indexToDrop = nodes.indexOf(elem);

                let splitWidthColumns = grid.style.gridTemplateColumns.split(' ');
                let value = parseFloat(splitWidthColumns[indexToDrop].replace('fr', ''));
                if (e.clientX - elementResizible.offsetLeft < 200) {
                    return;
                } else {
                    splitWidthColumns[indexToDrop] = `${e.clientX - elementResizible.offsetLeft}px`;
                }
                document.querySelector('.grid').style.gridTemplateColumns = splitWidthColumns.join(
                    ' '
                );
            };
            document.body.addEventListener('mousemove', mouseMove, false);
            document.body.addEventListener(
                'mouseup',
                e => {
                    document.body.removeEventListener('mousemove', mouseMove, false);
                },
                false
            );
        }
    }
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.container {
    width: 1280px;
    margin: 0 auto; 
}
.calculator {
    display: grid;
    width: 300px;
    margin: 0 auto;
    grid-template-columns: repeat(4, 1fr);
    grid-auto-col0s: minmax(50px, auto);
    .display,
    .key {
        padding-top: 25%;
    }
    .arrow {
        display: inline-block;
        vertical-align: middle;
        width: 0;
        height: 0;
        margin-left: 5px;
        opacity: 0.66;
    }
    .arrow.asc {
        border-left: 4px solid transparent;
        border-right: 4px solid transparent;
        border-bottom: 4px solid #fff;
    }
    .arrow.dsc {
        border-left: 4px solid transparent;
        border-right: 4px solid transparent;
        border-top: 4px solid #fff;
    }
    .display {
        grid-column: 1 / 5;
        background-color: #333;
        color: white;
    }
    .key {
        background-color: #f2f2f2;
        border: 1px solid #333;
    }
    .zero {
        grid-column: 1 / 3;
    }
    .operator {
        background-color: orange;
        color: white;
    }
}
.grid {
    display: grid;
    border: 1px solid red;
    max-width: 1280px;
    min-width: 5 * 200px;
    overflow-x: auto;
    .col {
        grid-row: 1 / 5;
        border-left: 1px solid red;
        min-width: 200px;
        overflow: hidden;
        &:hover {
            background-color: blue;
        }
        .row {
            border-bottom: 1px solid red;
            white-space: nowrap;
            height: 18px;
            &.header {
                position: relative;
                background-color: aqua;
                &.bananinha1::after {
                    
                }
                &.bananinha2::after {
                    content: 'bananinha2';
                }
                &.bananinha3::after {
                    content: 'bananinha3 bananinha3 bananinha3';
                }
                &.bananinha4::after {
                    content: 'bananinha4';
                }
                &.bananinha5::after {
                    content: 'bananinha5';
                }
                span {
                    position: absolute;
                    right: 0px;
                    height: 100%;
                    widows: 10px;
                    border: 1px solid #000;
                    &:hover {
                        cursor: w-resize;
                    }
                }
            }
            &:not(.header) {
                -webkit-touch-callout: none; /* iOS Safari */
                -webkit-user-select: none; /* Safari */
                -khtml-user-select: none; /* Konqueror HTML */
                -moz-user-select: none; /* Firefox */
                -ms-user-select: none; /* Internet Explorer/Edge */
                user-select: none; /* Non-prefixed version, currently supported by Chrome and Opera */
            }
        }
    }
}
</style>
