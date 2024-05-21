pip install ocr-nanonets-wrapper flatten_json json
from nanonets import NANONETSOCR
model = NANONETSOCR()
model.set_token('7514b653-993d-11ee-b34c-a61ebceb06aa')

import json
import pandas as pd

tables_json = model.convert_to_tables(input('Digite o caminho do arquivo PDF: '))
print(json.dumps(tables_json, indent=2))

arquivo_json = input('Digite o nome do arquivo JSON: ')

with open(arquivo_json, 'w', encoding='utf-8') as json_file:
    json.dump(tables_json, json_file, ensure_ascii=False, indent=2)

def json_to_excel(json_file_path, excel_file_path):
    with open(json_file_path, 'r') as file:
        data = json.load(file)

    table_data = {}
    page_offset = 0 # Armazena deslocamento das linhas baseado na página atual	

    for item in data:
        if 'prediction' in item:
            for prediction in item['prediction']:
                if 'cells' in prediction:
                    for cell in prediction['cells']:
                        row = cell['row'] + page_offset
                        col = cell['col']
                        text = cell.get('text', '')

                        if row not in table_data:
                            table_data[row] = {}
                        table_data[row][col] = text

                    page_offset += max([cell['row'] for cell in prediction['cells']]) 

    df = pd.DataFrame.from_dict(table_data, orient='index').sort_index(axis=0).sort_index(axis=1).fillna('')

    df.to_excel(excel_file_path, index=False)

json_file_path = arquivo_json
excel_file_path = input('Digite o caminho do arquivo Excel: ')

json_to_excel(json_file_path, excel_file_path)  
