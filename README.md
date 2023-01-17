# Хакатон УРФУ 2023
## Команда - DreamTeamOne
***Участники команды:***
- Крайнов Александр
- Спирова Анастасия
- Цинцов Никита
- Кожемяков Константин
- Пахарчук Андрей
## Проект - Автоматизация обработки машиночитаемых документов
### Описание задачи
На основе [набора данных](https://www.dropbox.com/sh/d5h5f3yrql8x392/AACQ2WYa5qYCqjC8QuVZ5TJ4a?dl=1)  нужно построить граф знаний о компании, который включает:
- Организационную структуру (список подразделений)
- Список сотрудников и их должностей
- Должностные обязанности и полномочия сотрудников

На основе построенного графа знаний нужно разработать систему, которая ищет ошибки в проверочных документах, например:
- Опечатки в названиях подразделений или ФИО сотрудников
- Несуществующие подразделения или сотрудники, ошибки в должностях
- Несоответствие тематики документа подразделению, в которое он направлен для исполнения (в бухгалтерию направлен документ с задачей по ИТ)

## Преобразование данных 
[Преобразование данных](https://github.com/Somertonman/hackathon_2023/blob/main/data_preparation.ipynb) в датафрейм происходит при помощи модели transformers

Поиск искомых полей в тексте происходит за счет парсинга текстового поля Задачи `task_responsibles_people`

```
  question = "Какая должность?"
  df['position'] = df.apply(lambda x: im_pad(qa_pipeline(question, x['task_responsibles_people'])['answer']), axis = 1)

  question = "Какое имя?"
  df['name'] = df.apply(lambda x: im_pad(qa_pipeline(question, x['task_responsibles_people'])['answer']), axis = 1)
```
Получаем читаемый датафрейм в виде таблице, в которой мы получили следующие поля:
- Номер Задачи `task_num`
- Текст задачи `task_text`
- Отдел `dept`
- Должность `position`
- Имя сотрудника `name`

<img width="1068" alt="Screenshot at Jan 18 00-21-48" src="https://user-images.githubusercontent.com/94981693/213028192-0c07654c-d8ae-4252-9feb-834f01ac9238.png">

## Построение графа связей
Для построения [графа связей](https://github.com/Somertonman/hackathon_2023/blob/main/graph_org.ipynb) использована библиотека [NetworkX](https://networkx.org/)
```
G = nx.Graph()

comp_name = 'COMPANY'

for z in list(departments):
    G.add_edge(comp_name, z)
    # iterate through the rows of the DataFrame
    for i, row in df.iterrows():
        # add task_responsibles_groups node
        dept = row["dept"]
        G.add_node(dept, size=df["dept"].value_counts()[dept])
        # add position node
        position = row["position"]
        G.add_node(position, size=df["position"].value_counts()[position])
        # add name node
        name = row["name"]
        G.add_node(name, size=df["name"].value_counts()[name])
        # add edges between task_responsibles_groups and position
        G.add_edge(dept, position)
        # add edges between position and name
        G.add_edge(position, name)
```

Граф отражает взаимосвязь отделов компании, должностей и конкретных сотрудников

Для отрисовки графа использована библиотека matplotlib

```
import matplotlib.pyplot as plt

plt.figure(figsize=(30,30)) # set the size of the figure
pos = nx.spring_layout(G) # use spring layout to calculate node positions
nx.draw(G, pos, with_labels=True)
```
![image_2023-01-18_01-10-45](https://user-images.githubusercontent.com/94981693/213029833-8f5ea06f-32ca-42b2-8cd3-b9e6f9118930.png)



