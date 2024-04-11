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

      <!-- 画面偏移用 -->
      <div :class="black_board"></div>

      <!-- 現在のプレイヤー -->
      <div class="now-player">
        <div :class="now_player_class+'card'">
          <div class="now-player-card-text">
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
    let bomb_point = ref(3)
    let now_bomb_status = ref("warning")
    let next_player_buff_point = ref(0)
    let now_player = ref(0)
    let player_data = ref([
      {p_id:0, p_name:"ひとし", hand:["g1","r1"]},
      {p_id:1, p_name:"おかざき", hand:[]},
    ])

    let wall_status = ref("wall-disactive")

    let now_player_class = ref("now-player-card-active ")

    let black_board = ref("black-board-disactive")

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
      changePlayer()
    }

    const changePlayer=()=>{
      if(bomb_point.value <=0){
        console.log("gameover/敗者: ",player_data.value[now_player.value].p_name)
        gameOver()
      }else{
        black_board.value = "black-board-active"
        console.log("playerチェンジ")
        now_player_class.value = "now-player-card-disactive "
        console.log(player_data.value.length)
        console.log("now_player:", now_player.value)
        setTimeout(() => {
          if(player_data.value.length == now_player.value+1){
            now_player.value = 0;
          }else{
            now_player.value+=1
          }
          now_player_class.value = "now-player-card-active "
          black_board.value = "black-board-disactive"
          wall_status.value = "wall-disactive"
        }, 600);
      }
    }

    //カードを引く
    const drawCard=()=>{
      if(player_data.value[now_player.value].hand.length == 2){
        alert("カードは引けません")
      }else{
        wall_status.value = "wall-active"
        player_data.value[now_player.value].hand.push("r1")
        setTimeout(() => {
          changeBombPoint()
        }, 1000);
      }
    }

    const gameOver=()=>{

    }

    // カード効果を記入
    const useCard=(card,index)=>{
      wall_status.value = "wall-active"
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
    }

    return{
      now_view,
      bomb_point,
      now_bomb_status,
      next_player_buff_point,
      player_data,
      now_player,
      wall_status,
      now_player_class,
      black_board,
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

  .now-player{
      width: 100%;
      position:absolute;
      top:0;
    .now-player-card-active{
      animation: slideIn 1.5s linear;
      margin: 1rem;
      .now-player-card-text{
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
    .now-player-card-disactive{
      display: none;
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
        animation: flipFadeIn 1.5s linear;
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

  @keyframes flipFadeIn {
    0%{
      opacity: 0;
      transform: rotateY(1550deg);
    }
    50%{
      opacity: 1;
      transform: rotateY(0deg);
    }
  }

  @keyframes upFadeOut {
    0%{
      opacity: 1;
      transform: translate(0);
    }
    50%{
      opacity: 0;
      transform: translate(0,2rem);
    }
  }

  .black-board-active{
    position: fixed;
    top:0;
    left:0;
    z-index:10000;
    width: 100vw;
    height: 100vh;
    background-color: black;
    opacity: 1;
    animation: blackBoardFadeIn 0.6s linear;
  }
  .black-board-disactive{
    animation: blackBoardFadeOut 0.6s linear;
  }
  @keyframes blackBoardFadeOut {
    0%{
      display: block;
      position: fixed;
      top:0;
      left:0;
      z-index:10000;
      width: 100vw;
      height: 100vh;
      background-color: black;
      opacity: 1;
    }
    99%{
      display: block;
      position: fixed;
      top:0;
      left:0;
      z-index:10000;
      width: 100vw;
      height: 100vh;
      background-color: black;
      opacity: 0;
    }
    100%{
      display: none;
    }
  }
  @keyframes blackBoardFadeIn {
    0%{
      display: block;
      position: fixed;
      top:0;
      left:0;
      z-index:10000;
      width: 100vw;
      height: 100vh;
      background-color: black;
      opacity: 0;
    }
    99%{
      display: block;
      position: fixed;
      top:0;
      left:0;
      z-index:10000;
      width: 100vw;
      height: 100vh;
      background-color: black;
      opacity: 1;
    }
    100%{
      display: block;
    }
  }
}
</style>
