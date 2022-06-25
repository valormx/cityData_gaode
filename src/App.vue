<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="处理高德地图获取的省市区数据"/>

  <button class="btn" @click="apply">点击开始处理</button>

</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import pinyin from 'js-pinyin'
import Http from 'axios'
// import fileSaver from 'file-saver'  //需要保存在json文件启用此行


export default {
  name: 'App',
  components: {
    HelloWorld
  },
  methods: {
    apply() {
      // console.log(dataJson);
      let cityData = {};
      let key = 'd9e27385778a19c55425dfaac84887e1';  //高德地图 Web服务key
      Http.get(`https://restapi.amap.com/v3/config/district?key=${key}&subdistrict=3`).then(response => {
        if (response.status === 200) {
          cityData = response.data.districts[0].districts
        } else {
          console.error('出错了');
          return;
        }
        let china = cityData;
        let newData = china.map(item => {
          return {
            adcode: item.adcode,
            name: item.name,
            // level: item.level,
            position: item.center,
            child: item.districts.map(v => {
              return {
                adcode: v.adcode,
                name: v.name,
                // level: v.level,
                position: v.center,
                province: item.name,
                child: v.districts.map(m => {
                  return {
                    adcode: m.adcode,
                    name: m.name,
                    // level: m.level,
                    position: m.center,
                  }
                })
              }
            })
          }
        })
        let temp = [];
        newData.forEach(item => {
          let tmp
          if (['710000', '810000', '820000'].includes(item.adcode)) { //台湾，香港，澳门特殊处理
            tmp = [item];
          } else if (['110000', '120000', '310000', '500000'].includes(item.adcode)) { //四大直辖市 北京 天津 上海 重庆
            let target = JSON.parse(JSON.stringify(item));
            let _tmp = JSON.parse(JSON.stringify(item.child));
            target.child = [];
            _tmp.forEach(item => {
              target.child = target.child.concat(item.child);
            })
            tmp = [target];
          } else {
            tmp = JSON.parse(JSON.stringify(item.child));
          }
          tmp.forEach(m => {
            m.child.unshift({
              adcode: item.adcode,
              name: '全城',
              // level: item.level,
              position: item.position,
            })
          })
          temp = temp.concat(tmp);
        })
        pinyin.setOptions({checkPolyphone: false, charCase: 0});

        temp.forEach(item => {
          item.letter = item.adcode === '500000' ? 'C' : pinyin.getCamelChars(item.name).slice(0, 1)
        })
        temp.sort(function (a, b) {
          if (a.letter > b.letter) return 1;
          else if (a.letter < b.letter) return -1;
          else {
            if (a.adcode > b.adcode) return 1;
            else if (a.adcode < b.adcode) return -1;
            else return 0
          }
        });
        let arr = [];
        for (let i = 0; i < 26; i++) {
          arr.push({
            letter: String.fromCharCode(65 + i),
            list: [],
          })
        }
        temp.forEach(item => {
          let index = arr.findIndex(v => v.letter === item.letter);
          if (index !== -1) {
            arr[index].list.push(item)
          }
        })
        arr = arr.filter(item => {
          return item.list.length > 0
        })
        console.log(arr);

        /**
         * 需要保存为json文件 启用下面三行
         */
        // let jsonData = JSON.stringify(arr);
        // let blob = new Blob([jsonData], {type: ''});
        // fileSaver.saveAs(blob, 'cityData.json')
      })

    }
  },
  created() {
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.btn {
  width: 200px;
  height: 40px;
  font-size: 18px;
  color: #fff;
  border: none;
  border-radius: 5px;
  background-color: #41b883;

  &:active {
    background-color: #328f66;
  }
}
</style>
