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
		<div class="grid" style="grid-template-columns:1fr 1fr 1fr 1fr 1fr">
			<div class="col col-1" draggable @dragstart="drag" @drop="drop" @dragover="dragover">
				<div class="row header header-1 bananinha1"><span @mousedown.prevent="resizeColumn"></span></div>
				<div class="row">data bananinha1</div>
				<div class="row">data bananinha1</div>
			</div>
			<div class="col col-2" draggable @dragstart="drag" @drop="drop" @dragover="dragover">
				<div class="row header header-2 bananinha2"><span @mousedown.prevent="resizeColumn"></span></div>
				<div class="row">data bananinha2</div>
				<div class="row">data bananinha2</div>
			</div>
			<div class="col col-3" draggable @dragstart="drag" @drop="drop" @dragover="dragover">
				<div class="row header header-3 bananinha3"><span @mousedown.prevent="resizeColumn"></span></div>
				<div class="row">data bananinha3</div>
				<div class="row">data bananinha3</div>
			</div>
			<div class="col col-4" draggable @dragstart="drag" @drop="drop" @dragover="dragover">
				<div class="row header header-4 bananinha4"><span @mousedown.prevent="resizeColumn"></span></div>
				<div class="row">data bananinha4</div>
				<div class="row">data bananinha4</div>
			</div>
			<div class="col col-5" draggable @dragstart="drag" @drop="drop" @dragover="dragover">
				<div class="row header header-5 bananinha5"><span @mousedown.prevent="resizeColumn"></span></div>
				<div class="row">data bananinha5</div>
				<div class="row">data bananinha5</div>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import Vue from 'vue';
import { setFlagsFromString } from 'v8';
import { setTimeout } from 'timers';

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
            columnDrag: ''
        };
    },
    created: function() {
        /* this.disableSelect(); */
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
                        console.log(domElement.innerHTML);
                        this.operator = domElement.innerHTML;
                        break;
                }
                return;
            }

            if (this.operator) {
                console.log(this.operator);
                this.numberTwo = `${this.numberTwo}${domElement.innerHTML}`;
                this.display = this.numberTwo;
                return;
            }

            this.numberOne = `${this.numberOne}${domElement.innerHTML}`;
            this.display = this.numberOne;

            console.log(`numberOne:${this.numberOne}`);
            console.log(`operator:${this.operator}`);
            console.log(`numberTwo:${this.numberTwo}`);
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
        drop(event: Event) {
            let columnDrop = event.srcElement.parentElement;
            let grid = columnDrop.parentElement;
            let nodes = Array.prototype.slice.call(grid.children);
            let indexToDrop = nodes.indexOf(columnDrop);
            let indexDrag = nodes.indexOf(this.columnDrag);

            console.log(indexDrag);
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
                /* console.log(`${document.querySelector('.grid .col-1').offsetWidth + e.clientX}`); */
                if (e.clientX < 200) {
                    return;
                } else {
                    console.log(elementResizible.offsetWidth - elementResizible.offsetLeft);
                    splitWidthColumns[indexToDrop] = `${elementResizible.offsetWidth -
                        elementResizible.offsetLeft}px`;
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
    /* width: 500px; */
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
    max-width: 1020px;
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
            &.header {
                position: relative;
                background-color: aqua;
                &.bananinha1::after {
                    content: 'bananinha1';
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
