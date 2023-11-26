<template>
  <div class="wrap">
    <table>
      <thead>
        <tr>
          <th @click="sortTable('o_id')">ID</th>
          <th @click="sortTable('client_name')">Имя клиента</th>
          <th @click="sortTable('diets')">Диета</th>
          <th @click="sortTable('tariff')">Тариф</th>
          <th @click="sortTable('address')">Адрес</th>
          <th @click="sortTable('phone')">Телефон</th>
          <th @click="sortTable('dates')">Дата</th>
          <th @click="sortTable('discount')">Дисконт</th>
          <th @click="sortTable('order_sum')">Сумма Заказа</th>
          <th @click="sortTable('order_payed')">Оплата</th>
          <th @click="sortTable('pay_status')">Статус</th>
          <th @click="sortTable('courier_comment')">Комментарий курьера</th>
          <th @click="sortTable('inner_comment')">Внутренний комментарий</th>
          <th>
            <div @click="sortTable('daysToEnd')">Длительность</div>
            <div>
              <select v-model="filterValue" @change="filterByStatus">
                <option value="Заканчивается сегодня">
                  Заканчивается сегодня
                </option>
                <option value="Не законченные">Не законченные</option>
                <option value="Законченные">Законченные</option>
              </select>
            </div>
          </th>
        </tr>
      </thead>
      <tbody>         
        <tr v-for="item in sortedData" :key="item.o_id">
          <td>{{ item.o_id }}</td>
          <td>{{ item.client_name }}</td>
          <td>
            <div v-for="(diet, index) in item.diets" :key="index">
              {{ diet }}
              <hr v-if="index !== item.diets.length - 1" />
            </div>
          </td>
          <td>{{ item.tariff.join(", ") }}</td>
          <td>{{ item.address }}</td>
          <td>{{ item.phone }}</td>
          <td>
            <div v-for="(date, index) in item.dates" :key="index">
              {{ date.start_date }} - {{ date.end_date }}
              <hr v-if="index !== item.dates.length - 1" />
            </div>
          </td>
          <td>{{ item.discount }}</td>
          <td>{{ item.order_sum }}</td>
          <td>{{ item.order_payed }}</td>
          <td>{{ item.pay_status }}</td>
          <td>{{ item.courier_comment }}</td>
          <td>{{ item.inner_comment }}</td>
          <td>{{ item.daysToEnd }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: [], // Здесь хранится исходный массив данных JSON
      sortedData: [], // Массив для отображения отсортированных данных
      sortOrder: 1, // Порядок сортировки (1 - по возрастанию, -1 - по убыванию)
      sortedColumn: null, // Столбец, по которому происходит сортировка
    };
  },
  async mounted() {
    try {
      const response = await fetch(".//public/data.json");
      this.data = await response.json();
      this.sortedData = this.data.slice();
    } catch (error) {
      console.error("Ошибка загрузки файла JSON:", error);
    }
    //вызываем метод который высчитывает дни до окончания диеты
    this.fetchOrders();
  },
  methods: {
    sortTable(column) {
      if (this.sortedColumn === column) {
        // Если столбец уже отсортирован, меняем порядок сортировки
        this.sortOrder *= -1;
      } else {
        // Если сортируем не отсортированный столбец, устанавливаем его в качестве текущего
        this.sortedColumn = column;
        this.sortOrder = 1; // Сортировка по умолчанию - по возрастанию
      }

      if (column === "daysToEnd") {
        column = "daysToEndValue"; // Используем числовое поле для сортировки
      }
      this.sortedData = this.data.slice().sort((a, b) => {
        // Сортировка данных в соответствии с выбранным столбцом и порядком сортировки
        // const valA = a[column];
        // const valB = b[column];
        const valA = a[column];
        const valB = b[column];

        if (valA < valB) {
          //Если значение valA меньше valB, возвращается -1 для указания, что a должно быть перед b. Умножение на this.sortOrder определяет направление сортировки (по возрастанию или убыванию).
          return -1 * this.sortOrder;
        }
        if (valA > valB) {
          //Если значение valA больше valB, возвращается 1 для указания, что b должно быть перед a. Умножение на this.sortOrder определяет направление сортировки.
          return 1 * this.sortOrder;
        } else if (valA === valB) {
          return 0;
        }
      });
    },

    async fetchOrders() {
      try {
        const response = await fetch(".//public/data.json");
        this.data = await response.json();
        // Вычисляем количество дней до начала каждого заказа
        this.data.forEach((order) => {
          const endDate = new Date(order.dates[0].end_date);
          const today = new Date();
          var daysDiff = Math.ceil((endDate - today) / (1000 * 60 * 60 * 24));
          order.daysToEndValue = daysDiff; // Добавляем числовое значение для сортировки

          const daysLabel = formatDaysLabel(Math.abs(daysDiff));

          if (daysDiff > 0) {
            order.daysToEnd = `Заканчивается через ${daysDiff} ${daysLabel}`;
          } else if (daysDiff < 0) {
            order.daysToEnd = `Закончилось ${Math.abs(
              daysDiff
            )} ${daysLabel} назад`;
          } else {
            order.daysToEnd = `Закончилось сегодня`;
          }
          //функция основывается на правилах русского языка для склонения числительных. Она выбирает правильное склонение слова "день" в зависимости от числа.
          function formatDaysLabel(number) {
            const cases = [2, 0, 1, 1, 1, 2];
            return ["день", "дня", "дней"][
              number % 100 > 4 && number % 100 < 20
                ? 2
                : cases[Math.min(number % 10, 5)]
            ];
          }
        });

        this.sortTable("daysToEnd");
      } catch (error) {
        console.error("Ошибка загрузки файла JSON:", error);
      }
    },
  },
  computed: {
    filteredData() {
      // фильтрует данные в соответствии с выбранной опцией фильтрации
      if (this.filterValue === "Заканчивается сегодня") {
        return this.data.filter(
          (item) => item.daysToEnd === "Заканчивается сегодня"
        );
      } else if (this.filterValue === "Не законченные") {
        return this.data.filter((item) => item.daysToEnd !== "Не законченные");
      } else if (this.filterValue === "Законченные") {
        return this.data.filter((item) => item.daysToEnd === "Законченные");
      } else {
        return this.data; // Если ни один из фильтров не выбран, возвращаем исходные данные
      }
    },
    sortedFilteredData() {
      //сортирует отфильтрованные данные так, чтобы строки, соответствующие выбранному фильтру, находились в верхней части массива данных.
      const filteredData = this.filteredData;
      const unfilteredData = this.data.filter(
        (item) => !filteredData.includes(item)
      ); // Оставшиеся строки

      // Сначала добавляем строки, удовлетворяющие фильтру, затем остальные
      return filteredData.concat(unfilteredData);
    },
  },
};
</script>

<style>
* {
  margin: 0;
  box-sizing: border-box;
}
.wrap {
  background-color: #bcbbbb;
  padding: 10px;
}
table {
  background-color: #fff;
  border-collapse: collapse;
  width: 100%;
}

th,
td {
  border: 1px solid #bcbbbb;
  text-align: center;
  vertical-align: middle;
  padding: 3px;
  font-size: 14px;
}
tr:nth-child(even) {
  background-color: #dddddd;
}

th {
  height: 60px;
  background-color: #333333;
  cursor: pointer;
  font-weight: 500;
  color: #fff;
}
th:nth-of-type(5),
th:nth-of-type(7),
th:nth-of-type(12),
th:nth-of-type(13) {
  width: 180px;
}
th div:first-child {
  border-bottom: 1px solid #bcbbbb;
  margin-bottom: 3px;
}
select {
  border: none;
  background-color: #333333;
  font-weight: 500;
  font-size: 12px;
  text-align: center;
  color: #fff;
}
</style>
