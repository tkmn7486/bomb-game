<template>
  <link href="https://fonts.googleapis.com/earlyaccess/nicomoji.css" rel="stylesheet">
  <div id="app">
    <!-- タイトル -->
    <div class="title-page" v-if="now_view == 'title'" @click="goSettingPage()">
      <h1>it is bomb!</h1>
    </div>

    <!-- ゲーム設定 -->
    <div class="setting-page" v-else-if="now_view == 'setting'">
      <button @click="StartGame()">ゲームスタート</button>
      <button @click="goTitlePage()">タイトルへ戻る</button>
    </div>

    <!-- ゲーム画面 -->
    <div class="game-page" v-else-if="now_view == 'game'">
      <!-- 誤操作を防ぐための透明な壁 -->
      <div :class="wall_status"></div>

      <!-- 現在のプレイヤー -->
      <div class="now_player">
        <div class="now_player-card card">
          <div class="now_player-card-text">
            <h1>{{ player_data[now_player].p_name }}</h1>
            <p>さんのばん</p>
          </div>
        </div>
      </div>

      <!-- ボムを表示する場所 -->
      <div class="bomb-place">
        <img src="./assets/bomb_normal.png" :class="'bomb-img-'+now_bomb_status">
      </div>

      <!-- プレイヤーの手札（イベントカード） -->
      <div class="player-hand">
        <div class="event-cards">
          <div class="event-card card" v-for="(card,index) in player_data[now_player].hand" :key="card.id" @click="useCard(card,index)">
            <img :src="require('./assets/cards/'+card+'.png')">
          </div>
        </div>
        <div class="pass-btn">
          <button class="btn btn-outline-dark" @click="drawCard()">カードを引く</button>
          <button class="btn btn-outline-dark" @click="changeBombPoint('normal')">つぎのひとへ</button>
        </div>
      </div>
    </div>

    <div class="error-page" v-else>
      エラーが発生しました。
      <button>タイトルへ戻る</button>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
export default {
  name: 'App',
  setup(){

    let now_view = ref("game")
    let bomb_point = ref(100)
    let now_bomb_status = ref("warning")
    let next_player_buff_point = ref(0)
    let now_player = ref(0)
    let player_data = ref([
      {p_id:0, p_name:"ひとし", hand:["g1","r1"]},
      {p_id:1, p_name:"おかざき", hand:[]},
    ])

    let wall_status = ref("wall-disactive")

    const goTitlePage=()=>{
      now_view.value = "title"
    }

    const goSettingPage=()=>{
      now_view.value = "setting"
    }

    const StartGame=()=>{
      now_view.value = "game"
      bomb_point.value = 100
    }

    const changeBombPoint=()=>{
      let point = 1+next_player_buff_point.value
      bomb_point.value = bomb_point.value - point
      console.log("ボムポイント：", bomb_point.value)
      next_player_buff_point.value = 0;
    }

    const changePlayer=()=>{
      console.log("playerチェンジ")
      wall_status.value = "wall-disactive"
    }

    //カードを引く
    const drawCard=()=>{
      if(player_data.value[now_player.value].hand.length == 2){
        alert("カードは引けません")
      }else{
        wall_status.value = "wall-active"
        setTimeout(() => {
          player_data.value[now_player.value].hand.push("r1")
          wall_status.value = "wall-disactive"
        }, 1000);
      }
    }

    // カード効果を記入
    const useCard=(card,index)=>{
      wall_status.value = "wall-active"
      setTimeout(() => {
        player_data.value[now_player.value].hand.splice(index,1)
        switch(card) {
          case "g1":
            console.log("g1のイベントカードを使用:", index);
            bomb_point.value += 1;
            changePlayer()
            break;

          case "r1":
            console.log("次のプレイヤーは消費ポイント+1");
            next_player_buff_point.value += 1;
            changePlayer()
            break;

          default:
            console.log("その他");
            changePlayer()
            break;
        }
      }, 1500);
    }

    return{
      now_view,
      bomb_point,
      now_bomb_status,
      next_player_buff_point,
      player_data,
      now_player,
      wall_status,
      goTitlePage,
      goSettingPage,
      StartGame,
      changeBombPoint,
      changePlayer,
      drawCard,
      useCard
    }
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

.title-page{
  position:absolute;
  top:0;
  left:0;
  height: 100vh;
  width: 100vw;
  h1{
    font-family: "Nico Moji";
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
}

.game-page{
  font-family: "Nico Moji";

  .wall-active{
    display: block;
    position: fixed;
    top: 0;
    left:0;
    width: 100vw;
    height: 100vh;
    z-index: 10000;
  }

  .wall-disactive{
    display: none;
  }

  .bomb-place{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    .bomb-img-normal{
      width: 60%;
      animation: expansion 2s linear infinite;
    }
    .bomb-img-warning{
      width: 100%;
      animation: expansion 0.5s linear infinite;
    }
  }

  @keyframes expansion {
    0% , 100%{
        scale: 1;
    }
    50%{
        scale: 1.2;
    }
  }

  .now_player{
      width: 100%;
      position:absolute;
      top:0;
    .now_player-card{
      animation: slideIn 1.5s linear;
      margin: 1rem;
      .now_player-card-text{
        h1{
          display: inline-block;
          margin:1rem;
          font-size: 3rem;
        }
        p{
          display: inline-block;
        }
      }
    }
  }

  @keyframes slideIn {
    0%{
      opacity: 0;
      transform: translate(-10rem,0);
    }
    50%{
      opacity: 1;
      transform: translate(0,0);
    }
  }

  .player-hand{
    width: 100%;
    position:absolute;
    bottom:0;
    padding: 1rem;
    .event-cards{
      margin: 0 auto;
      .event-card{
        width: 5rem;
        height: 7rem;
        margin: 1rem;
        display: inline-block;
        img{
          width: 4.8rem;
          height: 6.8rem;
          border: none;
          border-radius: 5px;
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
        }
      }
    }
    .pass-btn{
      button{
        margin: 0.5rem;
      }
    }
  }

}
</style>
