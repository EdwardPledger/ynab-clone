<template>
  <h2>Budget</h2>
  <div>
    <h4>Amount Left</h4>
    <input type="text" id="totalBudget">
  </div>
  <br>
  <button @click="saveBudget">Save Budget</button>
  <table id="budgetTable">
    <tr>
      <th>Category</th>
      <th>Budget</th>
      <th>Spent</th>
    </tr>
  </table>
  <button @click="addRow">Add Row</button>
  <div>
    <h4>Upload Budget</h4>
    <input type="file" id="uploadedBudget" accept="text/csv" @change="uploadBudget">
  </div>
</template>

<style>
  #budgetTable, th, td  {
    border: 1px solid black;
    border-collapse: collapse;
  }

  td {
    -webkit-user-modify: read-write;
    -moz-user-modify: read-write;
  }
</style>

<script>
import { defineComponent } from '@vue/composition-api'

export default defineComponent({
  setup() {
    
  },
  props: {
    totalAmount: Number
  },
  methods: {
    addRow() {
      const table = document.getElementById("budgetTable");
      const row = table.insertRow(1);
      const categoryCell = row.insertCell(0);
      const budgetCell = row.insertCell(1);
      const spentCell = row.insertCell(2);

      categoryCell.innerHTML = "Input category";
      budgetCell.innerHTML = 'Input budget';
      spentCell.innerHTML = '0';

      spentCell.addEventListener('change', this.updateTotalAmount);
    },
    saveBudget() {
      // Get the total amount
      const totalBudget = document.getElementById("totalBudget").value;

      // Get the budget items
      const budgetLines = [];
      const budgetTable = document.getElementById("budgetTable");
      const rowLength = budgetTable.rows.length;

      // Skip first row, just the table values
      for (var i=1; i<rowLength; i++) {
        const cells = budgetTable.rows.item(i).cells;
        const budgetLine = [];

        for (var j=0; j<cells.length; j++) {
          budgetLine.push(cells[j].innerHTML);
        }
        
        budgetLines.push(budgetLine);
      }

      // Create the csv file
      let csvContent = totalBudget + '\n';

      budgetLines.forEach(budgetLine => {
        csvContent += budgetLine.join(',');
        csvContent += '\n';
      });
      
      const csv = new Blob([csvContent], { type: "text/csv" });
      const csvUrl = URL.createObjectURL(csv);

      // Download the csv file
      const element = document.createElement('a');

      element.href = csvUrl;
      element.target = '_blank';
      element.download = 'budget.csv';

      element.click();
    },
    uploadBudget(e) {
      const file = e.target.files[0];
      const fileReader = new FileReader();

      fileReader.readAsText(file);

      fileReader.onload = () => {
        const csvContent = fileReader.result;
        const csvContentArray = csvContent.split(/\n/);
        
        // Attach values to budget amount and budget table
        document.getElementById("totalBudget").value = csvContentArray[0];

        const table = document.getElementById("budgetTable");

        // Fist line is the budget amount and last line is an exta new line
        for (var i = 1; i < csvContentArray.length - 1; i++) {
          const rowData = csvContentArray[i].split(',');

          const row = table.insertRow(1);
          row.insertCell(0).innerHTML = rowData[0];
          row.insertCell(1).innerHTML = rowData[1];
          row.insertCell(2).innerHTML = rowData[2];
        }
      }

      fileReader.onerror = () => console.log(fileReader.error);
    },
    updateTotalAmount() {
      let totalBudget = document.getElementById("totalBudget").value;
      totalBudget -= 100;
      document.getElementById("totalBudget").value = totalBudget;
    }
  }
})
</script>
