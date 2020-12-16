<template>
	<div @drop="drop($event)" @dragover="allowDrop($event)">
		<a-table class="drag-table" :columns="columns" :data-source="data" :pagination="false" :show-header="false"
						 ref="dragTable" size="small" bordered>
<!--			<template slot="dragElement" slot-scope="element">-->
<!--				<div v-if="element.isDraggable" :id="element.id" :key="element.id" @dragstart="drag($event)" draggable="true">-->
<!--					{{ element.text }}-->
<!--				</div>-->
<!--				<div v-else></div>-->
<!--			</template>-->
		</a-table>
	</div>
</template>

<script>


  const elements = [{row: 0, col: 0, span: 1, id: '1', text: '元素一'}, {row: 1, col: 0, span: 1, id: '2', text: '元素二'},
    {row: 2, col: 1, span: 2, id: '3', text: '元素三'}, {row: 3, col: 0, span: 2, id: '4', text: '元素四'},];

  export default {
    name: "DragTable",
    computed: {
      columns() {
        const columns = [];
        for (let i = 0; i < this.colLen; i++) {
          columns.push({
            title: i,
            dataIndex: i,
            key: i,
            align: 'center',
            scopedSlots: {customRender: 'dragElement'},
          })
        }
        //绘制合并的数据数组
        const arr = [];
        for (let i = 0; i < this.colLen; i++) {
          arr[i] = [];
          for (let j = 0; j < this.rowLen; j++) {
            arr[i][j] = 1;
          }
        }


        for (let i = 0; i < this.colLen; i++) {
          let temp = [];
          //找出某一列的
          for (let cell of this.cells) {
            if (cell.col === i) {
              temp.push(cell);
            }
          }
          for (let j = 0; j < this.rowLen; j++) {
            for (let cell of temp) {
              if (cell.row === j) {
                arr[i][j] = cell.span;
                for (let k = 1; k < cell.span; k++) {
                  arr[i + 1][j] = 0;
                }
              }
            }
          }
        }
        for (let i = 0; i < arr.length; i++) {
          let temp = arr[i];
          let column = columns[i];
          column.customRender = (value, row, index) => {
            const obj = {
              children: <div></div>,
              attrs: {},
            };
            if (value.isDraggable) {
              obj.children =
                  <div id={value.id} key={value.id} draggable="true" ondragstart={this.drag}>{value.text}</div>;
            }
            for (let j = 0; j < temp.length; j++) {
							if (index ===  j){
                obj.attrs.colSpan = temp[j];
							}
            }
            return obj;
          };


        }

        console.log(columns);
        return columns;
      },
      data() {
        const data = [];
        for (let i = 0; i < this.rowLen; i++) {
          data[i] = {};
          data[i].key = i;
          for (let j = 0; j < this.colLen; j++) {
            data[i][j] = {isDraggable: false};
          }
        }
        for (let cell of this.cells) {
          data[cell.row][cell.col] = {id: cell.id, text: cell.text, isDraggable: true};
        }
        return data;
      }
    },
    created() {
      this.cells = elements;
    },
    data() {
      return {cells: [], rowLen: 5, colLen: 5};
    },
    methods: {
      drop(event) {
        const id = event.dataTransfer.getData("text");
        if (id === "") {
          return;
        }
        const newTd = event.target.parentNode;
        const row = newTd.parentNode.rowIndex;
        const col = this.getColIndex(newTd);
        for (let cell of this.cells) {
          if (id === cell.id) {
            cell.col = col;
            cell.row = row;
            this.colLen = Math.max(this.colLen, cell.col + cell.span + 1);
            this.rowLen = Math.max(this.rowLen, cell.row + 2);
            break;
          }
        }
      },
      allowDrop(event) {
        event.preventDefault();
      },
      drag(event) {
        event.dataTransfer.setData("text", event.target.id);
        console.log("drag", event.target.id);
      },
      getColIndex(newTd) {
        if (newTd.previousElementSibling === null) {
          return newTd.cellIndex + newTd.colSpan - 1;
        }
        return this.getColIndex(newTd.previousElementSibling) + newTd.colSpan;
      },
    }
  }
  //
</script>

<style>


	div.drag-table tbody > tr > td {
		padding: 0 !important;
	}

	div.drag-table tbody > tr > td > div {
		height: 34px;
		/*width: 100px;*/
		padding: 8px;
	}

</style>