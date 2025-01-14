<template>
  <div id="app">
    <h1>Dynamic Table</h1>
    
    <input v-model="searchId" placeholder="Search by ID" />
    
    <label for="filter">Filter Value:</label>
    <input v-model="filterValue" type="number" placeholder="Enter number" />
    
    <div>
      <label for="itemsPerPage">Items per page:</label>
      <select v-model="itemsPerPage">
        <option value="10">10</option>
        <option value="20">20</option>
        <option value="50">50</option>
      </select>
    </div>

    <table>
      <thead>
        <tr>
          <th @click="sortTable('ID')">ID</th>
          <th @click="sortTable('baseMean')">baseMean</th>
          <th @click="sortTable('log2FoldChange')">log2FoldChange</th>
          <th @click="sortTable('lfcSE')">lfcSE</th>
          <th @click="sortTable('stat')">stat</th>
          <th @click="sortTable('pvalue')">pvalue</th>
          <th @click="sortTable('padj')">padj</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in filteredData" :key="item.ID">
          <td>{{ item.ID }}</td>
          <td>{{ item.baseMean }}</td>
          <td>{{ item.log2FoldChange }}</td>
          <td>{{ item.lfcSE }}</td>
          <td>{{ item.stat }}</td>
          <td>{{ item.pvalue }}</td>
          <td>{{ item.padj }}</td>
        </tr>
      </tbody>
    </table>

    <!-- 分页控件 -->
    <div>
      <button @click="currentPage = Math.max(currentPage - 1, 1)">Previous</button>
      <span>Page {{ currentPage }}</span>
      <button @click="currentPage = Math.min(currentPage + 1, totalPages)">Next</button>
    </div>
  </div>
</template>

<script>
import * as XLSX from 'xlsx'; // 导入 xlsx 库

export default {
  data() {
    return {
      data: [], // 存储从Excel文件加载的数据
      searchId: '', // 搜索ID的过滤器
      filterValue: '', // 数值过滤器
      itemsPerPage: 10, // 每页显示多少条记录
      currentPage: 1, // 当前页数
      sortKey: '', // 排序依据的列
      sortOrder: 'asc', // 排序顺序（升序或降序）
    };
  },
  computed: {
    // 计算过滤后的数据
    filteredData() {
      let filtered = this.data;

      // 应用搜索过滤器
      if (this.searchId) {
        filtered = filtered.filter(item => item.ID.toString().includes(this.searchId));
      }

      // 应用数值过滤器
      if (this.filterValue) {
        filtered = filtered.filter(item => item.baseMean > this.filterValue);
      }

      // 应用排序
      if (this.sortKey) {
        filtered.sort((a, b) => {
          const aValue = a[this.sortKey];
          const bValue = b[this.sortKey];
          return this.sortOrder === 'asc' ? aValue - bValue : bValue - aValue;
        });
      }

      // 分页
      const startIndex = (this.currentPage - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      return filtered.slice(startIndex, endIndex);
    },
    totalPages() {
      return Math.ceil(this.filteredData.length / this.itemsPerPage);
    }
  },
  methods: {
    // 从Excel文件获取数据
    fetchData() {
      // 使用axios加载public目录中的Excel文件
      fetch('/front-end-dynamic-table.xlsx')
        .then(response => response.arrayBuffer())
        .then(data => {
          const workbook = XLSX.read(data, { type: 'array' });
          const sheetName = workbook.SheetNames[0]; // 获取第一个工作表
          const sheet = workbook.Sheets[sheetName];
          const jsonData = XLSX.utils.sheet_to_json(sheet); // 将工作表数据转换为JSON

          this.data = jsonData; // 将数据赋值给 Vue 数据对象
        })
        .catch(error => {
          console.error('Error loading Excel file:', error);
        });
    },

    // 排序表格
    sortTable(key) {
      if (this.sortKey === key) {
        this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
      } else {
        this.sortKey = key;
        this.sortOrder = 'asc';
      }
    }
  },
  mounted() {
    this.fetchData(); // 在组件加载时获取数据
  }
};
</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}
table th, table td {
  padding: 8px;
  border: 1px solid #ddd;
}
table th {
  cursor: pointer;
}
</style>
