<template>
    <div class="balance-manager-main">
      <div class="input-group">
        <label for="bet-amount">Enter Bet Amount:</label>
        <input id="bet-amount" v-model.number="betAmount" type="number" min="1" :max="balance" />
      </div>
      
      <p class="balance">
        Balance: <span>{{ balance }}</span>
      </p>
  
      <p v-if="lastWonPrize !== null" class="won-prize">
        You won: <span>{{ lastWonPrize }}</span>
      </p>
  
      <button 
        @click="placeBet" 
        :disabled="betAmount <= 0 || betAmount > balance || !canSpin || rolling || countdown > 0" 
        class="spin-button"
      >
        Place Bet & Spin
      </button>
  
      <p v-if="!canSpin" class="message">
        You lost. Please refresh the page or update balance to spin again.
      </p>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      rolling: {
        type: Boolean,
        required: true
      },
      countdown: {
        type: Number,
        required: true
      }
    },
    data() {
      return {
        betAmount: 0,
        balance: this.getStoredBalance(),
        lastWonPrize: null, 
        canSpin: true 
      };
    },
    methods: {
      getStoredBalance() {
        const balance = localStorage.getItem('balance');
        return balance ? parseFloat(balance) : 1000; 
      },
      updateStoredBalance(newBalance) {
        localStorage.setItem('balance', newBalance);
        this.balance = newBalance;
      },
      placeBet() {
        if (this.betAmount <= 0 || this.betAmount > this.balance || !this.canSpin || this.rolling || this.countdown > 0) return;
  
        const newBalance = this.balance - this.betAmount;
        this.updateStoredBalance(newBalance);
  
        this.$emit('bet-placed', this.betAmount);
      },
  
      addPrizeToBalance(prizeAmount, prizeName) {
        if (prizeAmount === 0) {
          this.canSpin = false;
        } else {
          const newBalance = this.balance + prizeAmount;
          this.updateStoredBalance(newBalance);
        }
        this.lastWonPrize = prizeName;
      }
    }
  };
  </script>
  
  
  <style scoped> 
  .balance-manager-main {
    display: flex;
    flex-direction: column;
    gap: 20px;
    align-items: center;
    justify-content: space-between;
    padding: 20px;
    background-color: #f9f9f9;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    max-width: 400px;
    margin: 0 auto;
    min-height: 247px;
  }
  
  .input-group {
    display: grid;
    gap: 5px;
    grid-template-columns: 1fr 1fr;
    width: 100%;
    align-items: center;
  }
  
  .input-group label {
    font-size: 16px;
    color: #555;
    font-family: 'Arial', sans-serif;
  }
  
  .input-group input {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    font-size: 16px;
  }
  
  .balance-manager-main p {
    font-size: 18px;
    font-family: 'Arial', sans-serif;
    color: #333;
    margin: 0;
  }
  
  .balance span, .won-prize span {
    font-weight: bold;
    color: #4caf50;
  }
  
  .spin-button {
    padding: 10px 20px;
    font-size: 18px;
    color: white;
    background-color: #007bff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }
  
  .spin-button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
  
  .spin-button:hover:not(:disabled) {
    background-color: #0056b3;
  }
  
  .message {
    font-size: 16px;
    color: red;
  }
  
  .won-prize {
    color: #333;
    font-size: 18px;
  }
  
  .balance {
    color: #333;
    font-size: 18px;
  }
  </style>
  