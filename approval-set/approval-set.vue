<template>
  <div class="approval-set">
    <h2>审批流</h2>
    <div class="clear-flow-pop" @click="delNode">
      <i class="iconfont iconquxiao"></i>
    </div>
    <div class="add-flow-pop">
      <div class="add-flow-pop-content">
          <span @click="closePop(1)" class="add-flow-pop-content-clear">
            <i class="iconfont iconquxiao"></i>
          </span>
        <div class="add-flow-pop-content-wrap">
          <div class="item" @click="addNode(2)">
            <i class="iconfont iconshenpi"></i>
            <span>审批人</span>
          </div>
          <div class="item" @click="addNode(3)">
            <i class="iconfont iconchaosongwode-tianchong"></i>
            <span>抄送人</span>
          </div>
        </div>
      </div>
    </div>
    <div id="container"/>
  </div>
</template>

<script>
import G6 from '@antv/g6'

export default {
  name: 'approval-set',
  data () {
    return {
      g6Data: {}, // G6数据
      // 节点类型 1发起 2审批 3抄送
      nodeOptionalTypes: {
        1: 'launch',
        2: 'approval',
        3: 'copy'
      },
      graph: {}, // g6实例
      selAddNodeId: 0, // 点击添加节点按钮的id
      containerHeight: 800, // 容器高度
      selDelNodeId: 0// 删除节点id
    }
  },
  beforeMount () {
    this._initG6Data()
  },
  mounted () {
    this.initG6()
  },
  methods: {
    // 初始化节点信息
    _initG6Data () {
      const nodes = [this._g6GetNodeModelRect(1, '发起人')]
      // 只能依次加载
      this.g6Data = {
        nodes
      }
      this.g6Data.nodes.push(this._g6GetNodeImage())
      this.g6Data.nodes.push(this._g6GetNodeModelRect(2, '审批人'))
      this.g6Data.nodes.push(this._g6GetNodeImage())
      this.g6Data.nodes.push(this._g6GetNodeModelRect(3, '抄送人'))
      this.g6Data.nodes.push(this._g6GetNodeImage())
      this.g6Data.nodes.push({
        size: [90, 45],
        x: 200,
        y: this.getNodeModelRectNum[0] * 160 + 80,
        shape: 'ellipse',
        label: '结束',
        id: 'ellipse_end_1',
        style: {
          fill: '#E4E4E4',
          stroke: '#E4E4E4'
        }
      })
      this.g6Data.edges = this._g6GetEdges()
    },
    initG6 () {
      const graph = new G6.Graph({
        container: 'container',
        width: 800,
        height: this.containerHeight,
        nodeStyle: {
          default: {
            fill: '#40a9ff',
            stroke: '#096dd9'
          }
        },
        // 节点不同状态下的样式集合
        nodeStateStyles: {
          // 鼠标点击节点，即 click 状态为 true 时的样式
          click: {
            fill: '#f5f5ff'
          },
          // 鼠标 hover 上节点，即 hover 状态为 true 时的样式
          hover: {
            fill: '#f5f5ff'
          }
        },
        linkCenter: true, // 使边连入节点的中心
        defaultEdge: {
          style: {
            endArrow: true,
            lineWidth: 2,
            stroke: '#888'
          }
        }
      })
      // 监听鼠标点击节点
      graph.on('node:click', e => {
        // 先将所有当前有 click 状态的节点的 click 状态置为 false
        const clickNodes = graph.findAllByState('node', 'click')
        clickNodes.forEach(cn => {
          graph.setItemState(cn, 'click', false)
        })
        const nodeItem = e.item

        // 设置目标节点的 click 状态 为 true
        graph.setItemState(nodeItem, 'click', true)
        if (nodeItem._cfg.currentShape === 'image') { // 判断是否是添加按钮点击
          let addflowpop = document.getElementsByClassName('add-flow-pop')[0]
          addflowpop.style.display = 'block'
          addflowpop.style.top = nodeItem._cfg.model.y - 30 + 'px' // 添加按钮位置+160
          addflowpop.style.left = 250 + 'px'
          this.selAddNodeId = nodeItem._cfg.id
        } else if (nodeItem._cfg.currentShape === 'modelRect') { // 点击审批节点
          // 取消气泡
          this.closePop(1)
          // this.showClearBtn(nodeItem)
        } else { // 点击结束

        }
      })
      // 鼠标进入节点
      graph.on('node:mouseenter', e => {
        const nodeItem = e.item // 获取鼠标进入的节点元素对象
        graph.setItemState(nodeItem, 'hover', true) // 设置当前节点的 hover 状态为 true
        if (nodeItem._cfg.currentShape === 'modelRect') {
          this.showClearBtn(nodeItem)
        }
      })

      // 鼠标离开节点
      graph.on('node:mouseleave', e => {
        const nodeItem = e.item // 获取鼠标离开的节点元素对象
        graph.setItemState(nodeItem, 'hover', false) // 设置当前节点的 hover 状态为 false
        if (nodeItem._cfg.currentShape === 'modelRect') {
          let selType = nodeItem._cfg.id.split('_')[1]
          if (selType === this.nodeOptionalTypes['2'] || selType === this.nodeOptionalTypes['3']) { // 只要审批和抄送才能删除
            this.closePop(2)
            this.selDelNodeId = nodeItem._cfg.id
          }
        }
      })
      graph.read(this.g6Data)
      this.graph = graph
    },
    // 显示删除按钮
    showClearBtn (nodeItem) {
      let selType = nodeItem._cfg.id.split('_')[1]
      if (selType === this.nodeOptionalTypes['2'] || selType === this.nodeOptionalTypes['3']) { // 只要审批和抄送才能删除
        let addflowpop = document.getElementsByClassName('clear-flow-pop')[0]
        addflowpop.style.display = 'block'
        addflowpop.style.top = nodeItem._cfg.model.y + 18 + 'px' // 添加按钮位置+18
        addflowpop.style.left = 264 + 'px'
        this.selDelNodeId = nodeItem._cfg.id
      }
    },
    // 删除节点
    delNode () {
      // 得到添加按钮的所属nodes的下标
      let selNodeIndex = this.g6Data.nodes.findIndex(item => item.id === this.selDelNodeId)
      let newOverNodes = []
      this.g6Data.nodes.forEach((item, index) => {
        if (index < selNodeIndex) { // 小于就直接添加原node
          newOverNodes.push(item)
        } else if (index > selNodeIndex + 1) { // 排除删除的指定node和下面那个添加按钮
          let newItem = Object.assign({}, item)
          let idArr = newItem.id.split('_')
          newItem.id = idArr[0] + '_' + idArr[1] + '_' + (idArr[2] * 1 - 1)
          newItem.y -= 160
          newOverNodes.push(newItem)
        } else { // 指定node和下面那个添加按钮
        }
      })
      // 重新赋值
      // console.table(newOverNodes)
      this.g6Data.nodes = [...newOverNodes]
      this.g6Data.edges = this._g6GetEdges()
      this.graph.changeData(this.g6Data)
      this.closePop(2)
    },
    // 取消气泡框 1 为添加按钮 2 为删除按钮
    closePop (type) {
      let flowPop = ''
      if (type === 1) {
        flowPop = document.getElementsByClassName('add-flow-pop')[0]
      } else { flowPop = document.getElementsByClassName('clear-flow-pop')[0] }
      flowPop.style.display = 'none'
    },
    // 点击添加节点 2审批 3抄送
    addNode (type) {
      // 得到添加按钮的所属nodes的下标
      let selNodeIndex = this.g6Data.nodes.findIndex(item => item.id === this.selAddNodeId)
      let newOverNodes = []
      this.g6Data.nodes.forEach((item, index) => {
        if (index < selNodeIndex) { // 小于就直接添加原node
          newOverNodes.push(item)
        } else if (index === selNodeIndex) { // 等于就生成node添加到数组
          newOverNodes.push(item)
          let appoint_index = item.id.split('_')[2] * 1 + 1 // 当前点击添加按钮的下标+1
          newOverNodes.push(this._g6GetNodeModelRect(type, type === 2 ? '审批人' : '抄送人', appoint_index))
          newOverNodes.push(this._g6GetNodeImage(appoint_index))
        } else { // 大于的则重新排序 id+1 y+160
          let newItem = Object.assign({}, item)
          let idArr = newItem.id.split('_')
          newItem.id = idArr[0] + '_' + idArr[1] + '_' + (idArr[2] * 1 + 1)
          newItem.y += 160
          newOverNodes.push(newItem)
        }
      })
      // 判断最后一个y是否大于总高度
      if (newOverNodes[newOverNodes.length - 1].y > this.containerHeight) {
        this.containerHeight = newOverNodes[newOverNodes.length - 1].y + 100
        this.graph.changeSize(800, this.containerHeight)// 改变画布大小。
      }
      // 重新赋值
      // console.table(newOverNodes)
      this.g6Data.nodes = [...newOverNodes]
      this.g6Data.edges = this._g6GetEdges()
      this.graph.changeData(this.g6Data)
      this.closePop(1)
    },
    // 配置得到类型为ModelRect的node  appointIndex为指定下标 用于修改
    _g6GetNodeModelRect (type, label, appoint_index, description = '描述文本', x = 200) {
      let appIndex = 0
      if (appoint_index !== undefined) {
        appIndex = appoint_index
      } else {
        appIndex = this.getNodeModelRectNum[0] + 1
      }
      const theme = type === 1 ? '#576A95' : type === 2 ? '#FF943E' : '#3296FA'
      const id = 'modelRect_' + this.nodeOptionalTypes[type] + '_' + appIndex
      return {
        id,
        x,
        y: appIndex * 160 - 60,
        description,
        shape: 'modelRect',
        label,
        // 节点中icon配置
        logoIcon: {
        // 是否显示icon
          show: false
        },
        stateIcon: {
          show: false
        },
        style: {
          // fill: '#f5f5ff',
          stroke: theme, // 边框色
          lineWidth: 2
        },
        // 左侧矩形 preRect
        preRect: {
          fill: theme, // 填充色
          width: 10
        }
      }
    },
    // 得到添加按钮node appointIndex为指定下标 用于修改
    _g6GetNodeImage (appoint_index) {
      let appIndex = 0
      if (appoint_index !== undefined) {
        appIndex = appoint_index
      } else {
        appIndex = this.getNodeModelRectNum[1] + 1
      }
      return {
        id: 'image_add_' + appIndex,
        img: 'http://localhost:8080/img/add.png',
        shape: 'image',
        size: 40,
        x: 200,
        y: appIndex * 160 + 20
      }
    },
    // 依据节点生成边的数据
    _g6GetEdges () {
      let edges = []
      this.g6Data.nodes.forEach((item, index) => {
        if (index !== this.g6Data.nodes.length - 1) {
          let obj = {}
          obj.source = item.id
          obj.target = this.g6Data.nodes[index + 1].id
          edges.push(obj)
        }
      })
      return edges
    }
  },
  computed: {
    getNodeModelRectNum () {
      let sumArr = [0, 0] // 第一个代表modelRect节点数 第二个代表image节点数
      if ('nodes' in this.g6Data) { // 如果存在节点 则计算
        this.g6Data.nodes.forEach(item => {
          if (item.id.includes('modelRect')) { sumArr[0]++ }
          if (item.id.includes('image')) { sumArr[1]++ }
        })
      }
      return sumArr
    }

  }
}
</script>

<style lang="less" scoped>
  .approval-set {
    background: #fff;
    border: 1px solid #e8eaec;
    position: relative;
    &:hover {
      box-shadow: 0 1px 6px rgba(0, 0, 0, 0.2);
      border-color: #eee;
    }

    h2 {
      padding: 8px 0;
      text-indent: 28px;
    }
    .clear-flow-pop{
      display: none;
      z-index: 1;
      position: absolute;
      cursor: pointer;
    }
    .add-flow-pop {
      display: none;
      z-index: 1;
      position: absolute;
      box-shadow: 0px 0px 18px #aaa;
      border-radius: 10px;

      .add-flow-pop-content {
        position: relative;
        top: 0px;
        left: 0px;
        width: 250px;
        height: 150px;
        background: #fff;
        -moz-border-radius: 12px;
        -webkit-border-radius: 12px;
        border-radius: 12px;
        .add-flow-pop-content-wrap{
          display: flex;
          flex-direction: row;
          .item{
            flex: 1;
            text-align: center;
            display: flex;
            flex-direction: column;
            cursor: pointer;
            .iconfont{
              margin: 5px auto;
              font-size: 34px;
              width: 70px;
              height: 70px;
              line-height: 70px;
              border-radius: 70px;
              border: 1px solid #eee;
              &:hover{
                background-color: #2d8cf0;
                color: #fff;
              }
            }
            .iconshenpi{
              color: #FF943E;
            }
            .iconchaosongwode-tianchong{
              color: #3296FA;
            }
          }
        }
        .add-flow-pop-content-clear{
          display: inline-block;
          width: 94%;
          text-align: right;
          margin-top: 5px;
          cursor: pointer;
        }
        &:before {
          position: absolute;
          content: "";
          width: 0;
          height: 0;
          right: 100%;
          top: 65px;
          border-top: 10px solid transparent;
          border-right: 15px solid #fff;
          border-bottom: 10px solid transparent;
        }
      }
    }
  }

</style>
