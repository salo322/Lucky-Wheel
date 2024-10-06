<template>
    <div class="wheel-main-container">
      <div class="wheel-wrapper">
        <div class="wheel-pointer"></div>
        <div class="wheel-bg" :class="{freeze: freeze}" :style="`transform: rotate(${wheelDeg}deg)`">
          <div class="prize-list">
            <div class="prize-item-wrapper" v-for="(item,index) in prizeList" :key="index">
              <div class="prize-item" :style="`transform: rotate(${(360 / prizeList.length) * index}deg)`">
                <div class="prize-name">{{ item.name }}</div>
                <div class="prize-icon">
                  <img :src="item.icon" />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
  
     <div class="bottom-container">
      <div class="countdown" v-if="countdown > 0">
        Next spin in: {{ countdown }} seconds
      </div>
      <BalanceManager 
      ref="balanceManager" 
      @bet-placed="onBetPlaced" 
      :rolling="rolling" 
      :countdown="countdown" 
     />

     <Leaderboard :topWinnings="topWinnings" />
     </div>
  
      <audio ref="spinSound" src="public/sounds/spin.mp3"></audio>

    </div>
  </template>
  
  <script>
  import BalanceManager from './BalanceManager.vue';
  import Leaderboard from './Leaderboard.vue';
  
  export default {
    components: {
    BalanceManager,
    Leaderboard
},
    data() {
      return {
        freeze: false,
        rolling: false,
        wheelDeg: 0,
        betAmount: 0,  
        prizeNumber: 8,
        countdown: 0, 
        prizeListOrigin: [
          { name: "$10", value: 10 },
          { name: "$50", value: 50 },
          { name: "$100", value: 100 },
          { name: "$200", value: 200 },
          { name: "Lost", value: 0 },   
          { name: "$500", value: 500 },
          { name: "$1000", value: 1000 },
          { name: "Lost", value: 0 }
        ],
        topWinnings: []
      };
    },
    computed: {
      prizeList() {
        return this.prizeListOrigin.slice(0, this.prizeNumber);
      }
    },
    methods: {
      onBetPlaced(betAmount) {
        if (this.countdown > 0) return;  
  
        this.betAmount = betAmount;
        this.onClickRotate();
      },
      onClickRotate() {
        if (this.rolling) return;
  
        const audio = this.$refs.spinSound;
        audio.volume = 1.0;
        audio.muted = false;
  
        const result = Math.floor(Math.random() * this.prizeList.length);
        this.roll(result);
      },
      roll(result) {
        this.rolling = true;
  
        const audio = this.$refs.spinSound;
        audio.play().catch(error => console.error("Failed to play audio:", error));
  
        const { wheelDeg, prizeList } = this;
        this.wheelDeg =
          wheelDeg - (wheelDeg % 360) + 6 * 360 + (360 - (360 / prizeList.length) * result);
  
        setTimeout(() => {
          this.rolling = false;
          audio.pause();
          audio.currentTime = 0;
  
          const prizeValue = prizeList[result].value;
          const prizeName = prizeList[result].name;
  
          this.$refs.balanceManager.addPrizeToBalance(prizeValue, prizeName);
  
          if (prizeValue > 0) {
          console.log('Updating leaderboard with:', { name: prizeName, value: prizeValue });

          // Update topWinnings array
          this.updateLeaderboard({ name: prizeName, value: prizeValue });
        }

          this.startCountdown();
          
        }, 4500);
      },
      updateLeaderboard(newPrize) {
      // Add the new prize to topWinnings and sort
      const updatedWinnings = [...this.topWinnings, newPrize].sort((a, b) => b.value - a.value);

      // Store only the top 3 winnings
      this.topWinnings = updatedWinnings.slice(0, 3);

      // Save top winnings to localStorage
      localStorage.setItem('topWinnings', JSON.stringify(this.topWinnings));
    },
      startCountdown() {
        this.countdown = 10; 
  
        const countdownInterval = setInterval(() => {
          this.countdown--;
          if (this.countdown === 0) {
            clearInterval(countdownInterval);  
          }
        }, 1000); 
      }
    },
    mounted() {
    // Load the top winnings from localStorage when the component mounts
    const storedWinnings = localStorage.getItem('topWinnings');
    if (storedWinnings) {
      this.topWinnings = JSON.parse(storedWinnings);
    }
  }

  };
  </script>
  
  <style scoped>
  *{
    box-sizing: border-box;
  }
  .bottom-container{
    display: flex;
    flex-direction: column;
    gap: 10px;
    max-width: 304px;
  }
  .wheel-main-container{
    display: flex;
    gap: 40px;
    align-items: center;
  }
  .prize-list .prize-item-wrapper:nth-child(odd) .prize-item {
    background-color: #FD8B51;
    clip-path: polygon(100% 0, 0 0, 51% 100%);
  }
  
  .prize-list .prize-item-wrapper:nth-child(even) .prize-item {
    background-color: #CB6040;
    clip-path: polygon(100% 0, 0 0, 51% 100%);
  }
  .wheel-wrapper {
    width: 400px;
    height: 400px;
    position: relative;
  }
  .wheel-pointer {
    width: 40px;
    height: 40px;
    border-radius: 1000px;
    background: #257180;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
    line-height: 60px;
    z-index: 10;
    cursor: pointer;
    border: 2px dashed;
  }
  .wheel-pointer::after {
    content: "";
    position: absolute;
    top: -32px;
    left: 50%;
    border-style: solid;
    border-color: transparent transparent #257180;
    transform: translateX(-50%);
    border-width: 0 10px 36px;
  }
  .wheel-bg {
    width: 100%;
    height: 100%;
    border-radius: 1000px;
    overflow: hidden;
    transition: transform 4s ease-in-out;
    background: #FCDE70;
    border: 5px dashed #654520;
  }
  .wheel-bg.freeze {
    transition: none;
    background: red;
  }
  .prize-list {
    width: 100%;
    height: 100%;
    position: relative;
    text-align: center;
  }
  .prize-item-wrapper {
    position: absolute;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
    width: 166px;
    height: 200px;
  }
  .prize-item {
    width: 100%;
    height: 100%;
    transform-origin: bottom;
  }
  .prize-name {
    padding: 16px 0;
    color: #fff;
    font-weight: bold;
    font-size: 18px;
    font-family: sans-serif;
  }
  </style>
  