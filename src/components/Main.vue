<template>
  <div class="main">
    <Row class="headline">
      <Col span="24">
      <h1>百度文库文档转文字工具</h1>
      <p>版本号：1.0.0 | 最后更新：2017-6-15</p>
      <p>Powered by
        <a href="http://frederickwang.com/">Frederick Wang</a>
      </p>
      </Col>
    </Row>
    <Row class="search-bar">
      <Col span="24">
      <br/>
      <Input v-model="document_address" icon="ios-close-outline" placeholder="请输入一个百度文库文档的网址" @on-click="clear_input"></Input>
      <br/>
      <br/>
      <Button type="success" @click="start_conversion">开始转换</Button>
      <Button type="error" @click="reset">重置</Button>
      <Button type="info" :disabled="save_btn_disabled" :loading="save_btn_loading" @click="save_txt">保存txt</Button>
      </Col>
    </Row>
    <Row class="conversion-result">
      <Col span="24">
      <br/>
      <Card :bordered="false">
        <p slot="title">转换结果</p>
        <paragraph v-for="(item,index) in conversion_result" :item="item" :key="index">
        </paragraph>
      </Card>
      </Col>
    </Row>
  </div>
</template>

<script>
import FileSaver from 'file-saver'

export default {
  name: 'main',
  data () {
    return {
      document_address: '',
      conversion_result: [],
      save_btn_disabled: true,
      save_btn_loading: false,
      title: ''
    }
  },
  methods: {
    start_conversion: function () {
      this.clear_result()
      // 简单的处理。把电脑版的网址转化为Wap版的网址
      let wapVersionAddress = this.document_address.split('?')[0]
      if (wapVersionAddress.indexOf('wapwenku.baidu.com') === -1) {
        wapVersionAddress = wapVersionAddress.replace(/wenku.baidu.com/, 'wapwenku.baidu.com')
      }
      window.$.get(`http://${window.location.hostname}/proxy.php`, { address: wapVersionAddress }, (body) => {
        this.save_btn_disabled = false
        this.save_btn_loading = true
        //
        let firstPage = window.$('<div></div>')
        firstPage.html(body)
        this.title = window.$('title', firstPage).text().replace('_百度文库', '')
        let sumPage = window.$('.info', firstPage).text().match(/(\d*)页/)[1]
        let currentPage = 1
        window.$('.content', firstPage).find('.txt').each((index, element) => {
          let txt = String(window.$(element).text()).trim()
          this.conversion_result.push(txt)
        })
        currentPage++
        var fetchPage = () => {
          window.$.get(`http://${window.location.hostname}/proxy.php`, { address: `${wapVersionAddress}?pn=${currentPage}&pu=` }, (body) => {
            let newPage = window.$('<div></div>').html(body)
            window.$('.content', newPage).find('.txt').each((index, element) => {
              let txt = String(window.$(element).text()).trim()
              this.conversion_result.push(txt)
            })
            currentPage++
            if (currentPage <= sumPage) {
              fetchPage()
            } else {
              this.save_btn_loading = false
            }
          })
        }
        fetchPage()
      })
      // this.conversion_result = document.documentElement.outerHTML
    },
    clear_input: function () {
      this.document_address = ''
    },
    clear_result: function () {
      this.conversion_result.length = 0
      this.save_btn_disabled = true
      this.save_btn_loading = false
    },
    reset: function () {
      this.clear_input()
      this.clear_result()
    },
    save_txt: function () {
      let text = ''
      for (let element of this.conversion_result) {
        text += `${element}\n`
      }
      let blob = new Blob([text], { type: 'text/plain;charset=utf-8' })
      FileSaver.saveAs(blob, `${this.title}.txt`)
    }
  },
  components: {
    'paragraph': {
      props: ['item'],
      template: `<p>{{item}}</p>`
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.main {
  margin-top: 20px;
  margin-left: 15px;
  margin-right: 15px;
  margin-bottom: 20px;
}
</style>
