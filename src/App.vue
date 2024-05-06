<template>
  <div class="card">
    <div class="calculator">
      <div class="input-container">
        <input type="text" v-model="inputValue" @keyup.enter="handleEnter" />
      </div>
      <div class="buttons-container">
        <div class="row" v-for="row in buttonRows" :key="row">
          <button
            v-for="button in row"
            :key="button"
            @click="handleClick(button)"
            :class="{
              operator: isOperator(button),
              clear: button === 'C',
              accounting: isAccountingButton(button),
              digit: !isNaN(parseInt(button)) || button === '.',
            }"
          >
            {{ buttonLabel(button) }}
          </button>
        </div>
      </div>
    </div>
    <div class="history-container">
      <div
        class="history-item"
        v-for="(result, index) in calculationHistory"
        :key="index"
      >
        {{ result }}
      </div>
      <canvas ref="lineChart"></canvas>
    </div>
  </div>
</template>

<script>
import Chart from "chart.js/auto";

export default {
  name: "AccountingCalculator",
  data() {
    return {
      inputValue: "",
      calculationHistory: [],
      lineChart: null,
      buttons: [
        ["(", ")", "%", "C"],
        ["7", "8", "9", "+"],
        ["4", "5", "6", "-"],
        ["1", "2", "3", "*"],
        ["0", ".", "=", "/"],
        ["%", "Tx", "Dsc", "Tot"],
      ],
    };
  },
  computed: {
    buttonRows() {
      return this.buttons;
    },
  },
  mounted() {
    this.renderLineChart();
  },
  methods: {
    handleClick(button) {
      if (button === "=") {
        this.calculateResult();
      } else if (button === "C") {
        this.inputValue = "";
      } else if (button === "Tx") {
        this.inputValue += "* 1.10"; // 10% tax for example
      } else if (button === "Dsc") {
        this.inputValue += "* 0.90"; // 10% discount for example
      } else if (button === "Tot") {
        this.calculateTotal();
      } else {
        this.inputValue += button;
      }
    },
    isOperator(button) {
      return isNaN(parseInt(button)) && button !== ".";
    },
    isAccountingButton(button) {
      return (
        ["(", ")", "%"].includes(button) ||
        ["Tx", "Dsc", "Tot"].includes(button)
      );
    },
    calculateResult() {
      try {
        let result = eval(this.inputValue);
        if (!isNaN(result)) {
          this.inputValue = result;
          this.addToHistory(result);
        } else {
          this.inputValue = "Error";
        }
      } catch (error) {
        this.inputValue = "Error";
      }
    },
    calculateTotal() {
      try {
        let total = eval(this.inputValue);
        if (!isNaN(total)) {
          this.inputValue = total.toFixed(2); // Round to 2 decimal places
          this.addToHistory(total);
        } else {
          this.inputValue = "Error";
        }
      } catch (error) {
        this.inputValue = "Error";
      }
    },
    buttonLabel(button) {
      switch (button) {
        case "Tx":
          return "Tax";
        case "Dsc":
          return "Dsc";
        case "Tot":
          return "Total";
        default:
          return button;
      }
    },
    handleEnter() {
      this.calculateResult();
    },
    addToHistory(result) {
      if (this.calculationHistory.length === 10) {
        this.calculationHistory.pop(); // Remove the oldest result if history reaches maximum capacity
      }
      const index = this.calculationHistory.length + 1;
      const indexedResult = `${index}: ${result}`;
      this.calculationHistory.unshift(indexedResult); // Add the result with its index
      this.reindexHistory(); // Reindex the history after adding the new result
    },
    reindexHistory() {
      this.calculationHistory = this.calculationHistory.map(
        (result, index) => `${index + 1}: ${result.split(":")[1].trim()}`
      );
      this.renderLineChart(); // Re-render the line chart after reindexing
    },
    renderLineChart() {
      const labels = this.calculationHistory.map((_, index) => index + 1);
      const data = this.calculationHistory.map((result) =>
        parseFloat(result.split(":")[1].trim())
      );

      if (this.lineChart) {
        this.lineChart.destroy(); // Destroy the existing chart instance
      }

      this.lineChart = new Chart(this.$refs.lineChart, {
        type: "line",
        data: {
          labels,
          datasets: [
            {
              label: "Calculation History",
              data,
              fill: false,
              borderColor: "rgb(75, 192, 192)",
              tension: 0.1,
            },
          ],
        },
        options: {
          scales: {
            x: {
              title: {
                display: true,
                text: "Calculation Index",
              },
            },
            y: {
              title: {
                display: true,
                text: "Result",
              },
            },
          },
        },
      });
    },
  },
};
</script>

<style scoped>
.card {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: flex-start;
  width: fit-content;
  background-color: #2c3e50;
  border-radius: 10px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.3);
  padding: 20px;
}

.calculator {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.input-container {
  margin-bottom: 10px;
}

input[type="text"] {
  width: 240px;
  height: 60px;
  padding: 15px;
  border: 2px solid #3498db;
  border-radius: 8px;
  background-color: #ecf0f1;
  color: #333;
  font-size: 16px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.5);
}

.buttons-container {
  display: flex;
  flex-direction: column;
}

.row {
  display: flex;
}

button {
  width: 60px;
  height: 60px;
  margin: 5px;
  border: 2px solid transparent;
  border-radius: 8px;
  background-color: #34495e;
  color: white;
  font-size: 18px;
  cursor: pointer;
  transition: background-color 0.3s, border-color 0.3s;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.3);
}

button:hover {
  background-color: #2c3e50;
}

.operator {
  background-color: #3498db;
}

.clear {
  background-color: #e74c3c;
}

.accounting {
  background-color: #2ecc71;
}

.digit {
  background-color: #4caf50;
}

.digit:hover {
  background-color: #45a049;
}

.history-container {
  width: 200px;
  padding: 10px;
}

.history-item {
  background-color: #f2f2f2;
  border-radius: 5px;
  padding: 5px;
  margin-bottom: 5px;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
}
</style>
