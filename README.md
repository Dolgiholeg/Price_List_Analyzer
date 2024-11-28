Университет Urban аттестационная работа

import csv

import os

import re

import tabulate


def export_to_html(sorted_result):

    #     Метод экспортирует отсортированные данные в HTML-формат и выводит их в консоль.
    #
    #     Параметры
    #     sorted_result (list): Список данных для экспорта.
    #     Возвращаемое значение
    #     создаёт HTML-файл и выводит данные в консоль.

    #     Инициализируем HTML
    result = '''
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>Позиции продуктов</title>
        <style>           
        table {
        font-family: "Lucida Sans Unicode", "Lucida Grande", Sans-Serif;
        font-size: 14px;
        border-collapse: collapse;
        text-align: center;
        }
        th, td:first-child {
        background: #AFCDE7;
        color: white;
        padding: 10px 20px;
        }
        th, td {
        border-style: solid;
        border-width: 0 1px 1px 0;
        border-color: white;
        }
        td {
        background: #D8E6F3;
        }
        th:first-child, td:first-child {
        text-align: left;
        }
        </style>           
    </head>
    <body>
        <table>
            <tr>
                <th>Номер</th>
                <th>Название</th>
                <th>Цена</th>
                <th>Фасовка</th>
                <th>Файл</th>
                <th>Цена за кг.</th>
            </tr>
    '''
    index = 0  # нумеруем позиции
    for i in sorted_result:
        index += 1
        result += f'''
            <tr>
                <td>{index}</td>
                        <td>{i[1]}</td>
                        <td>{i[2]}</td>
                        <td>{i[3]}{' кг'}</td>
                        <td>{i[0]}</td>
                        <td>{i[4]}</td>
                    </tr>
                '''
    result += '''
    </table>
    </body>
    </html>
    '''
    with open('output.html', 'w', encoding='utf8') as file:
        file.write(result)
    # вывод в консоль:
    headers = ['№', 'Наименование', 'Цена', 'Вес', 'Цена за кг', 'Файл']
    results_num = [[i + 1] + res[1:] + [res[0]] for i, res in enumerate(sorted_result)]  # нумеруем строчки
    print(tabulate.tabulate(results_num, headers=headers, tablefmt='grid', stralign='center'))
    

    #     Создаём класс PriceMachine
    class PriceMachine:

        def __init__(self):
            self.data = []  # сохраняем полученные списки строк в список после применения условий и патернов
    
    def load_prices(self, file_path=''):

        #     Метод сканирует указанный путь к каталогу. Ищет файлы со словом price в названии.
        #     В файле ищет столбцы с названием товара, ценой и весом.
        #     Возвращает список из списков со строками удовлетворяющим патернам
        #
        #     :param directory: file path from user_input
        #     :return: self.data = [[filename, product, price, weight, price/kg)]

        for filename in os.listdir(file_path):
            if filename.endswith('.csv') and 'price' in filename.lower():
                with open(os.path.join(file_path, filename), 'r', newline='', encoding='utf-8') as file:
                    reader = csv.DictReader(file)
                    for row in reader:
                        for column in row:
                            if re.search(r'(товар|название|наименование|продукт)', column, re.IGNORECASE):
                                product = row[column].strip()  
                            elif re.search(r'(розница|цена)', column, re.IGNORECASE):
                                price = float(row[column].replace(',', '.').strip())  
                            elif re.search(r'(вес|масса|фасовка)', column, re.IGNORECASE):
                                weight = float(row[column].replace(',', '.').strip())  
                        if product:
                            self.data.append([filename, product, price, weight, round(price / weight, 2)])
        
    def _search_product_price_weight(self, headers):

        #     Метод ищет переданное слово от пользователя в списке от load_price(self.data)
        #     Возвращает отсортированный результат поиска на экспорт и в консоль
        #     :param input_text: value(text) from user_input_find_text
        #     :return: sorted_results

        results = []
        for row in self.data:
            if re.search(headers, row[1], re.IGNORECASE):  # поиск по столбцу с названием
                results.append(row)
        sorted_results = sorted(results, key=lambda x: x[4])  # сортировка по цене за кг
        export_to_html(sorted_results)  # передаем отсортированный результат на экспорт

    def search_engine(self, input_text):

        # метод ищет переданное слово от пользователя в списке от load_price(self.data)
        # Возвращает отсортированный результат поиска на экспорт и в консоль
        # :param input_text: value(text) from find_text
        # :return: sorted_results

        results = []
        for row in self.data:
            if re.search(input_text, row[1], re.IGNORECASE):  # поиск по столбцу с названием
                results.append(row)
        sorted_results = sorted(results, key=lambda x: x[4])  # сортировка по цене за кг
        export_to_html(sorted_results)  # передаем отсортированный результат на экспорт

    def find_text(self, file_path):

        #     Метод запрашивает у пользователя ввод и выполняет поиск по данным.
        #     Параметры
        #     file_path (str): Путь к каталогу с файлами.
        #     Возвращаемое значение
        #     None
        #     Функция осуществляет ввод пользователя и передает искомое значение в search_engine(поисковик)
        #     А так же принимает абсолютный путь к папке с файлами и передает в load_prices(загрузчик)
        #     :param file_path: file path
        #     :return: value(text)

        self.load_prices(file_path)  # передаем путь папки в load_prices(загрузчик)

        while True:
            find_text_value = input('Что найти ("exit" чтобы закончить работу) ?: \n')
            if find_text_value.lower() == 'exit':
                print('the end')
                break
            self.search_engine(find_text_value)  # передаем введённое слово в поисковик


    # Создаем экземпляр класса PriceMachine и запускает основной процесс.


    if __name__ == '__main__':
    
        pm = PriceMachine()
        
        local_directory = os.path.dirname(os.path.abspath(__file__))  # абсолютный путь к файлам
        
        pm.find_text(local_directory)
    
        
    pm = PriceMachine()
    
    print(pm.load_prices())
    
    print(export_to_html())  
