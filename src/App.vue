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

      <!-- 現在のプレイヤー -->
      <div class="now_player">
        <div class="now_player-card card">
          <div class="now_player-card-text">
            <h3>たけし</h3>
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
          <div class="event-card card">
            イベント
          </div>
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
    let now_bomb_status = ref("normal")
    let player_data = ref([])

    const goTitlePage=()=>{
      now_view.value = "title"
    }

    const goSettingPage=()=>{
      now_view.value = "setting"
    }

    const StartGame=()=>{
      now_view.value = "game"
    }

    return{
      now_view,
      now_bomb_status,
      player_data,
      goTitlePage,
      goSettingPage,
      StartGame
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
  .bomb-place{
    position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    .bomb-img-normal{
      width: 60%;
      animation: expansion 2s linear infinite;
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
        h3{
          display: inline-block;
          margin:1rem;
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
      .event-card{
        width: 5rem;
        height: 7rem;
        margin: 0 auto;
      }
    }
  }

}
</style>
