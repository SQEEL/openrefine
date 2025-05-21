
## Создаём проект plan_sales

### Импортируем

- Ставим галочку игнорировать строку со значением 1
- Ставим галочку parse next со значением 2
- Называем проект plan_sales

![img](img/import_settings.png)

### Избавляемся от подытогов и некоторых колонок

В самом начале лучше переключить отображение на rows:

![img](img/switch_into_rows.png)

Можно по разному избавиться от группировок, но самый простой способ от убрать строки в которых нет id:

- Выбираем ID > Facet > Text facet
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

- Выбираем на колонку Column > Transpose > Transpose cells across columns into rows
![img](img/choose_transpose.png)

- Во всплывающем окне проставляем:

```
  Key column = Канал продаж
  Value column = Значение
```

![img](img/transpose_settings.png)

- На выходе получаем:
![img](img/view_after_transpose.png)

### Преобразовываем транспонированную таблицу


- Выбираем по очереди колонки "ID", "Наименование" и протягиваем (Edit cells > Fill down)

![img](img/id_fill_down.png)

- Создаём колонку Дата на основе колонки "Канал продаж" (Edit column > Add column base on this column)

![img](img/create_date_column.png)

- Вы всплывающем окне проставляем формулу и нажимаем ОК:

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
  2) Пишем текст "итого" (важно чтобы галочка case sensative не стояла)
  ![img](img/sales_channel_text_filter_total.png)
  3) Выбираем в разделе All > Edit rows > Remove matching columns
  ![img](img/sales_channel_text_filter_total_remove_matching_rows.png)
  4) Закрываем фильтр по Каналу продаж
  ![img](img/sales_channel_text_filter_total_quit.png)

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





