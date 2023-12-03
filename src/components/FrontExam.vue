<!-- FrontExam.vue -->
<template>
  <div>
    <div class="container">
      <h1>성적 입력 표</h1>
      <form @submit.prevent="submitGrade">
        <label for="subject">과목명:</label>
        <input type="text" id="subject" v-model="subject" required>

        <label for="credit">학점:</label>
        <input type="number" id="credit" v-model.number="credit" required>

        <label for="isRequired">필수:</label>
        <select id="isRequired" v-model="isRequired" required>
          <option value="필수">필수</option>
          <option value="선택">선택</option>
        </select>

        <label for="type">이수:</label>
        <select id="type" v-model="type" required>
          <option value="교양">교양</option>
          <option value="전공">전공</option>
        </select>

        <label for="attendance">출석점수:</label>
        <input type="number" id="attendance" v-model.number="attendance" min="0" max="20" required>

        <label for="assignment">과제점수:</label>
        <input type="number" id="assignment" v-model.number="assignment" min="0" max="20" required>

        <label for="midterm">중간고사:</label>
        <input type="number" id="midterm" v-model.number="midterm" min="0" max="30" required>

        <label for="finalExam">기말고사:</label>
        <input type="number" id="finalExam" v-model.number="finalExam" min="0" max="30" required>

        <button type="submit">추가</button>
      </form>
    </div>

    <div class="container">
      <h2>입력된 성적</h2>
      <button @click="clearLocalStorage">Local Storage 초기화</button>
      <button @click="deleteSelectedRows">선택된 행 삭제</button>
      <table v-if="gradeList.length > 0" class="grades-table">
        <thead>
          <tr>
            <th>선택</th>
            <th v-for="col in columns" :key="col.id">{{ col.name }}</th>
            
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(entry, index) in gradeList"
            :key="entry.id"
            :class="{ 'even-row': index % 2 === 0, 'odd-row': index % 2 !== 0 }"
          >
            <td>
              <input type="checkbox" v-model="entry.checked">
            </td>
            <td v-for="col in columns" :key="col.id">{{ formatCell(entry[col.id], col.type) }}</td>
          </tr>
          <tr>
            <td colspan="4">합계</td>
            <td>{{ totalCredit }}</td>
            <td>{{ totalAttendance }}</td>
            <td>{{ totalAssignment }}</td>
            <td>{{ totalMidterm }}</td>
            <td>{{ totalFinalExam }}</td>
            <td>{{ totalTotal }}</td>
            <td>{{ averageTotal.toFixed(2) }}</td>
            <td>{{ calculateLetterGrade(averageTotal) }}</td>
          </tr>
        </tbody>
      </table>
      <p v-else>입력된 성적이 없습니다.</p>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      subject: '',
      credit: null,
      isRequired: '필수',
      type: '교양',
      attendance: null,
      assignment: null,
      midterm: null,
      finalExam: null,
      selectedRow: null,
      showTotals: false,
      gradeList: [],
      totalCredit: 0,
      totalAttendance: 0,
      totalAssignment: 0,
      totalMidterm: 0,
      totalFinalExam: 0,
      totalTotal: 0,
      averageTotal: 0,
    };
  },
  computed: {
    columns() {
      return [
        { id: 'subject', name: '과목명', type: 'text' },
        { id: 'isRequired', name: '필수', type: 'text' },
        { id: 'type', name: '이수', type: 'text' },
        { id: 'credit', name: '학점', type: 'number' },
        { id: 'attendance', name: '출석점수', type: 'number' },
        { id: 'assignment', name: '과제점수', type: 'number' },
        { id: 'midterm', name: '중간고사', type: 'number' },
        { id: 'finalExam', name: '기말고사', type: 'number' },
        { id: 'total', name: '총점', type: 'number' },
        { id: 'average', name: '평균', type: 'number' },
        { id: 'grade', name: '성적', type: 'text' },
      ];
    },
  },
  methods: {
    submitGrade() {
      const total = this.attendance + this.assignment + this.midterm + this.finalExam;
      const average = total / 4;
      const grade = this.calculateLetterGrade(total);

      const newEntry = {
        id: Date.now(),
        subject: this.subject,
        credit: this.credit,
        isRequired: this.isRequired,
        type: this.type,
        attendance: this.attendance,
        assignment: this.assignment,
        midterm: this.midterm,
        finalExam: this.finalExam,
        total,
        average,
        grade,
      };

      this.gradeList.push(newEntry);
      
      this.updateTotals();

      localStorage.setItem('grades', JSON.stringify(this.gradeList));
    },
    clearLocalStorage() {
      localStorage.removeItem('grades');
      this.gradeList = [];
      this.updateTotals();
    },
    deleteSelectedRows() {
    this.gradeList = this.gradeList.filter((entry) => !entry.checked);
    localStorage.setItem('grades', JSON.stringify(this.gradeList));
    this.updateTotals();
    this.selectAll = false;
    },
    updateTotals() {
      this.totalCredit = this.gradeList.reduce((acc, entry) => acc + entry.credit, 0);
      this.totalAttendance = this.gradeList.reduce((acc, entry) => acc + entry.attendance, 0);
      this.totalAssignment = this.gradeList.reduce((acc, entry) => acc + entry.assignment, 0);
      this.totalMidterm = this.gradeList.reduce((acc, entry) => acc + entry.midterm, 0);
      this.totalFinalExam = this.gradeList.reduce((acc, entry) => acc + entry.finalExam, 0);
      this.totalTotal = this.gradeList.reduce((acc, entry) => acc + entry.total, 0);
      this.averageTotal = this.gradeList.length > 0 ? this.totalTotal / this.gradeList.length : 0;
    },
    calculateLetterGrade(total) {
      if (total === 100) return 'A+';
      else if (total >= 91) return 'A0';
      else if (total >= 81) return 'B+';
      else if (total >= 71) return 'B0';
      else if (total >= 61) return 'C+';
      else if (total >= 51) return 'C0';
      else if (total >= 41) return 'D+';
      else if (total >= 31) return 'D0';
      else return 'F';
    },
    formatCell(value, type) {
      if (type === 'number') {
        return value.toFixed(2);
      } else {
        return value;
      }
    },
  },
};
</script>



<style scoped>
/* 여기에 스타일링을 추가할 수 있습니다. */

.grades-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.grades-table th, .grades-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}

.grades-table th {
  background-color: #3498db;
  color: white;
}

.even-row {
  background-color: #f0f0f0;
}

.odd-row {
  background-color: #f9f9f9;
}

/* 체크박스 열에 대한 스타일 */
.grades-table input[type="checkbox"] {
  margin: 0; /* 기본 마진 제거 */
}
</style>