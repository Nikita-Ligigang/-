# -
Тестирование Ростелеком
from flask import Flask, url_for, request, render_template_string
import pandas as pd

app = Flask(__name__)

# Главная страница
@app.route('/')
def home():
    return '''
    <html>
        <head>
            <style>
                body {
                    display: flex;
                    flex-direction: column;
                    justify-content: center;
                    align-items: center;
                    height: 100vh;
                    margin: 0;
                    font-family: Arial, sans-serif;
                    background-image: url("''' + url_for('static', filename='Ross.jpg') + '''");
                    background-size: cover;
                    background-position: center;
                    background-attachment: fixed;
                }
                .content {
                    text-align: center;
                    background-color: rgba(255, 255, 255, 0.8);
                    padding: 20px;
                    border-radius: 10px;
                    color: black;
                }
                .button {
                    margin-top: 20px;
                    padding: 10px 20px;
                    border: none;
                    border-radius: 5px;
                    background-color: purple;
                    color: white;
                    cursor: pointer;
                }
                .button:hover {
                    background-color: darkviolet;
                }
                .logo {
                    margin-bottom: 20px;
                    max-width: 200%;
                    max-height: 100px;
                    width: auto;
                    height: auto;
                }
            </style>
        </head>
        <body>
            <div class="content">
                <img class="logo" src="''' + url_for('static', filename='logo.png') + '''" alt="Логотип">
                <h1>Добро пожаловать на Тестирование!</h1>
                <p>Нажмите для прохождения теста.</p>
                <button class="button" onclick="window.location.href='/about'">Начать</button>
            </div>
        </body>
    </html>
    '''

# Страница с тестом
@app.route('/about', methods=['GET', 'POST'])
def about():
    try:
        # Читаем Excel файл без заголовков
        df = pd.read_excel('testa.xlsx', header=None, names=['ID', 'Content', 'IsCorrect'])

        questions = []
        current_question = None

        # Обрабатываем строки
        for _, row in df.iterrows():
            # Если номер вопроса существует - это новый вопрос
            if not pd.isna(row['ID']):
                current_question = {
                    'number': int(row['ID']),
                    'text': row['Content'],
                    'answers': [],
                    'correct_answer': None
                }
                questions.append(current_question)
            else:
                # Добавляем ответы к текущему вопросу
                if current_question:
                    is_correct = not pd.isna(row['IsCorrect'])
                    answer = {
                        'text': row['Content'],
                        'correct': is_correct
                    }
                    current_question['answers'].append(answer)

                    # Запоминаем правильный ответ
                    if is_correct:
                        current_question['correct_answer'] = row['Content']

        if request.method == 'POST':
            # Обработка ответов
            correct_count = 0
            for q in questions:
                selected_answer = request.form.get(f'question_{q["number"]}')
                if selected_answer:  # Проверяем, что ответ выбран
                    if selected_answer == q['correct_answer']:
                        correct_count += 1
                else:
                    # Если ответ не выбран, считаем это за ошибку
                    pass  # Не увеличиваем correct_count, т.е. считаем за ошибку

            # Проверка проходного балла
            if correct_count >= 15:
                result = "Вы прошли тест!"
                color = "green"
            else:
                result = f"Вы не прошли тест. Правильных ответов: {correct_count} из {len(questions)}"
                color = "red"

            # Для фиолетового цвета надписи
            color = "purple"

            return f'''
            <html>
                <head>
                    <style>
                        body {{
                            display: flex;
                            flex-direction: column;
                            justify-content: center;
                            align-items: center;
                            height: 100vh;
                            margin: 0;
                            font-family: Arial, sans-serif;
                            background-image: url("{url_for('static', filename='Ross.jpg')}");
                            background-size: cover;
                            background-position: center;
                            background-attachment: fixed;
                        }}
                        .content {{
                            text-align: center;
                        }}
                        .result {{
                            font-size: 48px;
                            font-weight: bold;
                            color: {color};
                            margin-bottom: 20px;
                        }}
                        .home-button {{
                            padding: 10px 20px;
                            border: none;
                            border-radius: 5px;
                            background-color: purple;
                            color: white;
                            cursor: pointer;
                            font-size: 16px;
                        }}
                        .home-button:hover {{
                            background-color: darkviolet;
                        }}
                    </style>
                </head>
                <body>
                    <div class="content">
                        <h1 class="result">{result}</h1>
                        <button class="home-button" onclick="window.location.href='/'">На главную страницу</button>
                    </div>
                </body>
            </html>
            '''

        # Генерация формы для теста
        test_html = '''
        <html>
            <head>
                <style>
                    body {
                        font-family: Arial, sans-serif;
                        margin: 0;
                        padding: 0;
                        background-image: url("''' + url_for('static', filename='Ross.jpg') + '''");
                        background-size: cover;
                        background-position: center;
                        background-attachment: fixed;
                        color: black;
                        min-height: 100vh;
                    }
                    .question-block {
                        margin: 40px auto;
                        padding: 20px;
                        border: 2px solid purple;
                        border-radius: 10px;
                        background-color: rgba(255, 255, 255, 0.8);
                        max-width: 800px;
                    }
                    .question-number {
                        color: purple;
                        font-size: 1.2em;
                        margin-bottom: 15px;
                    }
                    .question-text {
                        color: black;
                    }
                    .answer {
                        margin: 10px 0;
                        padding: 12px;
                        border: 1px solid #ddd;
                        border-radius: 5px;
                        cursor: pointer;
                        transition: all 0.3s;
                        color: black;
                    }
                    label {
                        color: black;
                    }
                    .submit-button-container {
                        text-align: center;
                        margin-top: 20px;
                    }
                    .submit-button {
                        padding: 10px 20px;
                        border: none;
                        border-radius: 5px;
                        background-color: purple;
                        color: white;
                        cursor: pointer;
                    }
                    .submit-button:hover {
                        background-color: darkviolet;
                    }
                </style>
            </head>
            <body>
                <form method="post">
        '''

        for q in questions:
            test_html += f'''
            <div class="question-block">
                <div class="question-number">Вопрос {q['number']}</div>
                <div class="question-text">{q['text']}</div>
                <div class="answers">
            '''

            for answer in q['answers']:
                test_html += f'''
                <input type="radio" id="{answer['text']}" name="question_{q['number']}" value="{answer['text']}" required>
                <label for="{answer['text']}">{answer['text']}</label><br><br>
                '''

            test_html += "</div></div>"

        test_html += '''
                <div class="submit-button-container">
                    <button type="submit" class="submit-button">Завершить тест</button>
                </div>
                </form>
            </body>
        </html>
        '''
        return test_html

    except Exception as e:
        return f'''
        <div style="color: red; padding: 20px;">
            Ошибка загрузки теста: {str(e)}<br>
            Проверьте структуру файла test.xlsx:
            <ul>
                <li>Столбец A - номера вопросов</li>
                <li>Столбец B - вопрос и варианты ответов</li>
                <li>Столбец C - отметка правильного ответа (1)</li>
            </ul>
        </div>
        '''

if __name__ == '__main__':
    app.run(debug=False) # Важно!
