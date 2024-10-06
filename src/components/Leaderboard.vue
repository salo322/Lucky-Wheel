<template>
  <div class="leaderboard">
    <h3>Top 3 Winnings</h3>
    <ol>
      <li v-for="(winning, index) in topWinnings" :key="index">
        {{ winning.name }} - ${{ winning.value }}
      </li>
    </ol>
  </div>
</template>

<script>
export default {
  props: {
    topWinnings: {
      type: Array,
      default: () => []
    }
  }
};
</script>

  
  <style scoped>
  .leaderboard {
    padding: 10px 15px;
    background-color: #f9f9f9;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    text-align: center;
  }
  
  .leaderboard h3 {
    margin-bottom: 10px;
    font-size: 20px;
    color: #333;
  }
  
  .leaderboard ol {
    list-style: none;
    padding: 0;
  }
  
  .leaderboard li {
    font-size: 18px;
    color: #4caf50;
    font-weight: bold;
  }
  </style>
  