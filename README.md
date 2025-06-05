
> ℹ️ **Важно:** перед тем, как импортировать данные в OpenRefine,  
> сохраните файл с *уже пересчитанными итоговыми значениями*.  
> В движок OpenRefine не входит Excel-пересчёт, поэтому формулы должны быть
> превращены в числа заранее.

> ℹ️ **Информация:** если что-то зависло/перестали работать формулы или фильтры в UI -
> ничего страшного, так бывает с ним - просто обновите страницу (можно нажать на F5/кнопка рефреш/стать
> курсором на url и нажать enter), как правило изменения сохранены и после обновления можно продолжить практически с 
> того места где зависло. Также стоит учитывать, что он поднят на машине с минимальными параметрами оперативки и 
> процессоров, без каких либо дополнительных конфигурационных настроек.

### Скачайте подготовленный файл

[![Скачать «План продаж.xlsx»](https://img.shields.io/badge/📥-Скачать-2596be?style=for-the-badge&logo=Microsoft%20Excel&logoColor=white)](https://raw.githubusercontent.com/SQEEL/openrefine/main/excel/%D0%9F%D0%BB%D0%B0%D0%BD%20%D0%BF%D1%80%D0%BE%D0%B4%D0%B0%D0%B6.xlsx)

<details>
<summary>💻 или через <span style="font-family:monospace">wget</span>/<span style="font-family:monospace">curl</span></summary>

```bash
# wget
wget -O "План продаж.xlsx" \
  "https://raw.githubusercontent.com/SQEEL/openrefine/main/excel/%D0%9F%D0%BB%D0%B0%D0%BD%20%D0%BF%D1%80%D0%BE%D0%B4%D0%B0%D0%B6.xlsx"

# curl
curl -L -o "План продаж.xlsx" \
  "https://raw.githubusercontent.com/SQEEL/openrefine/main/excel/%D0%9F%D0%BB%D0%B0%D0%BD%20%D0%BF%D1%80%D0%BE%D0%B4%D0%B0%D0%B6.xlsx"
```
</details>

## Создаём проект `plan_sales`

- Заходим на страницу с приложением на сайт https://sqeel.space
- Выбираем раздел "Загрузить данные из / На этом компьютере"

![img](img/create_project.png)

### Импортируем в Openrefine с локального компьютера

- Ставим галочку "Не учитывать первые" со значением 1
- Ставим галочку "Парсить следующие" со значением 2
- Называем проект plan_sales

![img](img/import_settings.png)

### Избавляемся от подытогов и некоторых колонок

В самом начале лучше переключить отображение на **строк**:

![img](img/switch_into_rows.png)

Можно по разному избавиться от группировок, но самый простой способ от убрать строки в которых нет id:

- Выбираем ID > Фасет > Текстовый фасет
![img](img/id_filter_text_facet.png)

- Фильтруем по (blank)
![img](img/id_filtered_with_blanks.png)

- Удаляем выбранные строки
![img](img/remove_rows_id_filtered_with_blanks.png)

- Закрываем фильтр по ID
![img](img/close_id_filtered_with_blanks.png)

- Удаляем колонки 1,2 (с другими разберёмся после транспонирования)
![img](img/remove_column_1.png)

### Транспонируем

- Выбираем на колонку Column > Преобразование > Переместить ячейки колонок в строки
![img](img/choose_transpose.png)

- Во всплывающем окне проставляем:

```
  Колонка ключа = Канал продаж
  Колонка значения = Значение
```

![img](img/transpose_settings.png)

- На выходе получаем:
![img](img/view_after_transpose.png)

### Преобразовываем транспонированную таблицу


- Выбираем по очереди колонки "ID", "Наименование" и протягиваем (Правка ячеек > Размножить)

![img](img/id_fill_down.png)

- Создаём колонку Дата на основе колонки "Канал продаж" (Правка колонки > Добавить колонку основываясь на текущей...)

![img](img/create_date_column.png)

Во всплывающем окне проставляем:
- наименование "Дата"
- формулу и нажимаем ОК:

```
if(
  value.contains("ИТОГО"),
  value
     .partition("ИТОГО")[0]
     .trim()
     .toDate("MMM yy"),
  null
)
```
![img](img/date_formula.png)

где:
```
  value                    // ← исходная ячейка
     .partition("ИТОГО")[0]//  берём левую часть после "сплита" массива
     .trim()               //  убираем пробелы
     .toDate("MMM yy"),    //  превращаем в дату
  null                     //  где «ИТОГО» нет — оставляем пусто
```

- Протягиваем колонку Дата (Edit cells > Fill down):
![img](img/date_fill_down.png)

- Удаляем строки в которых Канал продаж с ИТОГО

  1) Выбираем Text filter 
  ![img](img/sales_channel_text_filter.png)
  2) Пишем текст "итого" (важно чтобы галочка "учитывать регистр" не стояла)
  ![img](img/sales_channel_text_filter_total.png)
  3) Выбираем в разделе Все > Правка строк > Удалить все отобранные записи
  ![img](img/sales_channel_text_filter_total_remove_matching_rows.png)
  4) Закрываем фильтр по Каналу продаж
  ![img](img/sales_channel_text_filter_total_quit.png)


- Переименовываем колонки, которые из-за одинакового названия получили хвост "__{порядковый номер}"

  1) Выбираем в колонке "Канал продаж" раздел Правка ячеек > Преобразование
  ![img](img/transform_same_column_menu.png)
  2) Пишем формулу с заменой по маске "__{число}" на пустую строку:

  ```
  value.replace(/__\d+$/,"")
  ```
    ![img](img/transform_same_column_expression.png)
  3) Получаем очищенный и корректные данные
  ![img](img/final_view_before_lookup_column.png)


## Создаём проект plan_sales_filter



### Импортируем

- Ставим галочку parse next со значением 1
- Называем проект plan_sales_filter

![img](img/import_plan_sales_settings.png)

- Создаём проект

## Возвращаемся в проект plan_sales

### Создаём колонку ID канала

- Создаём колонку на основе Канала продаж
![img](img/add_new_column_based_on_sales_channel.png)

- Прописываем название
```
cell.cross("plan_sales_filter", "Канал")[0].cells["ID_канала"].value
```
![img](img/plan_sales_id_channel_value.png)

- Получаем итоговую таблицу
![img](img/result.png)





