<template>
  <link href="https://fonts.googleapis.com/earlyaccess/nicomoji.css" rel="stylesheet">
  <div id="app">
    <!-- タイトル -->
    <div class="title-page" v-if="now_view == 'title'" @click="goSettingPage()">
      <h1>がめんをタッチ</h1>
    </div>

    <!-- ゲーム設定 -->
    <div class="setting-page" v-else-if="now_view == 'setting'">
      <h2>ゲームさんかしゃ：{{ player_data.length }}にん</h2>
      <div class="player-input">
        <div>
          <input v-model="p.p_name" type="text" placeholder="ひらがな5もじいない" v-for="p in player_data" :key="p.id">
        </div>
        <br>
        <br>
        <button class="btn btn-light" @click="addPlayer()">ついか+</button>
      </div>
      <button class="btn btn-lg btn-light start-btn" @click="StartGame()">ゲームスタート</button>
    </div>

    <!-- ゲーム画面 -->
    <div class="game-page" v-else-if="now_view == 'game'">
      <!-- 誤操作を防ぐための透明な壁 -->
      <div :class="wall_status"></div>

      <!-- 画面偏移用 -->
      <div :class="black_board"></div>

      <!-- 敗北プレイヤー -->
      <div :class="gameover">
        <h2>{{ player_data[now_player].p_name }}さん</h2>
        <h3>のまけ</h3>
        <button class="btn btn-lg btn-light" @click="now_view = 'title';">タイトルへもどる</button>
      </div>

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
          <button class="btn btn-lg btn-light" @click="drawCard()">カードをひく</button>
          <button class="btn btn-lg btn-light" @click="changeBombPoint('normal')">つぎのひとへ</button>
        </div>
      </div>
    </div>

    <div class="error-page" v-else>
      エラーがはっせいしました。
      <button>タイトルへもどる</button>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
export default {
  name: 'App',
  setup(){

    let now_view = ref("title")
    let bomb_point = ref(100)
    let now_bomb_status = ref("normal")
    let next_player_buff_point = ref(0)
    let now_player = ref(0)
    let player_data = ref([])

    let card_list = ref([
      "g1","g2","g3","r1","r2","r3","r10","y1"
    ])

    let wall_status = ref("wall-disactive")

    let now_player_class = ref("now-player-card-active ")

    let black_board = ref("black-board-disactive")

    let gameover = ref("gameover-false")

    const goTitlePage=()=>{
      now_view.value = "title"
    }

    const goSettingPage=()=>{
      now_view.value = "setting"
    }

    const StartGame=()=>{
      if(player_data.value.length == 0 || player_data.value.length == 1){
        alert("参加者が不足しています")
      }else{
        let data_issue = false
        for(let i=0; i<player_data.value.length; i++){
          player_data.value[i].p_id = i
          player_data.value[i].hand = []
          if(player_data.value[i].p_name==""){
            data_issue = true
          }
        }
        if(data_issue == true){
          alert("参加者の名前が抜けています")
        }else{
          gameover.value='gameover-false'
          wall_status.value='wall-disactive'
          now_bomb_status.value = "normal"
          now_view.value = "game"
          bomb_point.value = 100
        }
      }
    }

    const changeBombPoint=()=>{
      let point = 1+next_player_buff_point.value
      bomb_point.value = bomb_point.value - point
      console.log("ボムポイント：", bomb_point.value)
      next_player_buff_point.value = 0;
      changePlayer()
    }

    const addPlayer=()=>{
      player_data.value.push({p_id:0, p_name:"", hand:[]})
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
          if(bomb_point.value<=15){
            now_bomb_status.value = "warning"
          }else{
            now_bomb_status.value = "normal"
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
        alert("手札は2枚以上持つことができません。")
      }else{
        wall_status.value = "wall-active"
        let draw_index = Math.floor( Math.random() * card_list.value.length );
        player_data.value[now_player.value].hand.push(card_list.value[draw_index])
        setTimeout(() => {
          next_player_buff_point.value = next_player_buff_point.value+2;
          changeBombPoint()
        }, 1000);
      }
    }

    const gameOver=()=>{
      gameover.value = "gameover-true"
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

          case "g2":
            console.log("g2のイベントカードを使用:", index);
            bomb_point.value += 2;
            changePlayer()
            break;

          case "g3":
            console.log("g3のイベントカードを使用:", index);
            bomb_point.value += 3;
            changePlayer()
            break;
          
          case "r1":
            console.log("次のプレイヤーは消費ポイント+1");
            next_player_buff_point.value += 1;
            changePlayer()
            break;

          case "r2":
            console.log("次のプレイヤーは消費ポイント+2");
            next_player_buff_point.value += 2;
            changePlayer()
            break;

          case "r3":
            console.log("次のプレイヤーは消費ポイント+3");
            next_player_buff_point.value += 3;
            changePlayer()
            break;

          case "r10":
            console.log("次のプレイヤーは消費ポイント+10");
            next_player_buff_point.value += 10;
            changePlayer()
            break;

          case "y1":
            console.log("消費ポイントなしで次の人に回す");
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
      gameover,
      card_list,
      goTitlePage,
      goSettingPage,
      StartGame,
      changeBombPoint,
      changePlayer,
      drawCard,
      useCard,
      addPlayer
    }
  }
}
</script>

<style lang="scss">
html{
  background-color: black;
}
#app {
  font-family: "Nico Moji";
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.title-page{
  color: white;
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
    animation: blingText 1.5s linear infinite;
  }
}

@keyframes blingText {
    0%{
      opacity: 1;
    }
    50%{
      opacity: 0;
    }
    100%{
      opacity: 1;
    }
  }

.setting-page{
  background-color: black;
  color: white;
  .player-input{
    input{
      width: 9rem;
      margin: 1rem;
    }
  }
  .start-btn{
    position: fixed;
    bottom: 1rem;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
  }
}

.game-page{
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
      animation: sway 2s linear infinite;
    }
    .bomb-img-warning{
      width: 100%;
      animation: expansion 0.3s linear infinite;
      filter: invert(15%) sepia(95%) saturate(6932%) hue-rotate(358deg) brightness(75%) contrast(112%);
    }
  }

  @keyframes sway {
    0%{
      transform: rotate(0deg);
    }
    5%{
      transform: rotate(10deg);
    }
    15%{
      transform: rotate(-10deg);
    }
    25%{
      transform: rotate(10deg);
    }
    35%{
      transform: rotate(-10deg);
    }
    45%{
      transform: rotate(10deg);
    }
    55%{
      transform: rotate(-10deg);
    }
    65%{
      transform: rotate(10deg);
    }
    70%{
      transform: rotate(0deg);
    }
    100%{
      transform: rotate(0deg);
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
      transform: rotateY(100deg);
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

.gameover-false{
  display: none;
}

.gameover-true{
  color: red;
  display: block;
  position: fixed;
  top:0;
  left:0;
  width: 100vw;
  height: 100vh;
  background-color: black;
  z-index: 100000;
  animation: backBoardFadeIn 1.0s linear;
}
</style>
