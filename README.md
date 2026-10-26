# Harnessing-the-Power-of-Deception-in-Attack-Graph-Based-Security-Games
проработка статьи в резюме, коде и схемах





# Реализация обмана в играх безопасности на графах атак (Attack Graph Deception Game)

Этот репозиторий содержит практический разбор и программную реализацию математической модели децепции (обмана) на графах атак по мотивам исследования Carnegie Mellon University и Army Research Laboratory: *«Harnessing the Power of Deception in Attack Graph-Based Security Games»*.

---

## 📖 1. Теоретический конспект (Paper Review)

### Архитектура модели (The Game Structure)
Игра моделируется как **байесовская игра Стекльберга (Bayesian Stackelberg Game)** на ориентированном ациклическом графе атак (DAG) $G = (N, E)$.
* **Защитник (Лидер):** Ходит первым. Он изменяет физический граф сети, тратя фиксированный бюджет на обманные действия. Его цель — минимизировать математическое ожидание ущерба.
* **Атакующий (Последователь):** Ходит вторым. Проводит разведку, формирует свое видение графа и выбирает путь к цели, максимизирующий его выгоду.
* **Типы атакующих ($\Theta$):** Злоумышленники делятся на типы $\theta \in \Theta$ с априорной вероятностью $p_\theta$. Тип определяет уровень навыков разведки. Профессионалы с высокой вероятностью способны распознать ловушки защитника.

### Пространство стратегий обмана (Deception Actions)
Защитник искажает реальный граф $G$, создавая для атакующего «воспринимаемый граф» $G'_\theta$:
1. **Маскировка (Hiding):** Скрытие реального ребра/уязвимости. Атакующий не идет по этому пути, так как считает, что прохода нет.
2. **Приманки (Fake paths / Honeypots):** Создание ложных узлов и путей. Попадая туда, хакер тратит ресурсы, но гарантированно блокируется.
3. **Манипуляция ценностью (Altering payoffs):** Маскировка критически важного сервера под второстепенный (или наоборот).

Авторы доказывают, что для общего графа эта задача является **NP-трудной**. 
## 📊 2. Топология графа: Реальность vs Обман

Для генерации интерактивной схемы используется скрипт на Python (`visualize.py`). Он наглядно демонстрирует разницу между реальной топологией сети и тем, как её видит дезинформированный атакующий.

### Скрипт визуализации
```python
import matplotlib.pyplot as plt
import networkx as nx

# Инициализируем подложку для двух графиков
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 6))

# --- Левая панель: Реальный граф сети (Взгляд защитника) ---
G_true = nx.DiGraph()
G_true.add_edges_from([
    ('Start', 'Web Server'), 
    ('Web Server', 'DB (Real)'), 
    ('Web Server', 'Honeypot')
])

pos = {
    'Start': (0, 1), 
    'Web Server': (1, 1), 
    'DB (Real)': (2, 1.6), 
    'Honeypot': (2, 0.4)
}

# Отрисовка узлов
nx.draw_networkx_nodes(G_true, pos, nodelist=['Start', 'Web Server'], node_color='#a0cbe2', node_size=2000, ax=ax1)
nx.draw_networkx_nodes(G_true, pos, nodelist=['DB (Real)'], node_color='#8cd17d', node_size=2000, ax=ax1)
nx.draw_networkx_nodes(G_true, pos, nodelist=['Honeypot'], node_color='#ff9d9a', node_size=2000, ax=ax1)

# Отрисовка ребер (путь к реальной БД пунктирный - он скрыт)
nx.draw_networkx_edges(G_true, pos, edgelist=[('Start', 'Web Server')], edge_color='gray', width=2, arrowsize=20, ax=ax1)
nx.draw_networkx_edges(G_true, pos, edgelist=[('Web Server', 'DB (Real)')], edge_color='#499894', style='dashed', width=2, arrowsize=20, ax=ax1)
nx.draw_networkx_edges(G_true, pos, edgelist=[('Web Server', 'Honeypot')], edge_color='#e15759', width=2, arrowsize=20, ax=ax1)

nx.draw_networkx_labels(G_true, pos, font_size=10, font_weight='bold', ax=ax1)
ax1.set_title("Реальный граф (Защитник)\n[Путь к DB скрыт маскировкой]", fontsize=12, fontweight='bold', pad=15)
ax1.axis('off')

# --- Правая панель: Искаженный граф (Взгляд атакующего-новичка) ---
G_perceived = nx.DiGraph()
G_perceived.add_edges_from([
    ('Start', 'Web Server'), 
    ('Web Server', 'Honeypot')
])

# Отрисовка узлов (ловушка выделена золотым цветом как сверхценная)
nx.draw_networkx_nodes(G_perceived, pos, nodelist=['Start', 'Web Server'], node_color='#a0cbe2', node_size=2000, ax=ax2)
nx.draw_networkx_nodes(G_perceived, pos, nodelist=['Honeypot'], node_color='#f28e2b', node_size=2400, ax=ax2)

# Отрисовка ребер
nx.draw_networkx_edges(G_perceived, pos, edgelist=[('Start', 'Web Server')], edge_color='gray', width=2, arrowsize=20, ax=ax2)
nx.draw_networkx_edges(G_perceived, pos, edgelist=[('Web Server', 'Honeypot')], edge_color='#f28e2b', width=3, arrowsize=20, ax=ax2)

# Корректируем метку для отображения ценности ловушки в глазах хакера
labels_perceived = {'Start': 'Start', 'Web Server': 'Web Server', 'Honeypot': 'Honeypot\n(⭐ Сверхценно!)'}
nx.draw_networkx_labels(G_perceived, pos, labels=labels_perceived, font_size=9, font_weight='bold', ax=ax2)
ax2.set_title("Искаженный граф (Атакующий)\n[Видит только фейковый путь-приманку]", fontsize=12, fontweight='bold', pad=15)
ax2.axis('off')

plt.tight_layout()
plt.savefig("attack_graph_deception.png", dpi=300)
plt.show()
```

Запустив этот скрипт, вы получите файл `attack_graph_deception.png`, который можно вставить в описание репозитория:

![Топология графа атак](attack_graph_deception.png)
## 💻 3. Симуляция на Python

Скрипт ниже моделирует принятие решений атакующими двух типов: **Новичок (Low-skill)** и **Профи (High-skill)** в условиях, когда защитник применил стратегии маскировки и создания ложных целей.

### Зависимости
Для запуска проекта вам понадобятся:
```bash
pip install networkx matplotlib
```

### Код симуляции (`simulation.py`)
```python
import networkx as nx

class DeceptionGame:
    def __init__(self):
        # Реальная ценность серверов для ИБ-инфраструктуры
        self.real_payoffs = {
            'DB_Real': 100,      # Главная критическая цель
            'Honeypot': 0        # Ловушка-пустышка
        }
        
        # Стоимость прохода по ребрам (затраты атакующего на эксплойты)
        self.edge_costs = {
            ('Start', 'Web'): 10,
            ('Web', 'DB_Real'): 40,
            ('Web', 'Honeypot'): 15
        }

    def simulate_attacker(self, attacker_type):
        """
        Моделирование логики атакующего согласно его типу (навыкам разведки)
        """
        print(f"\n=== Анализ поведения атакующего: [{attacker_type.upper()}] ===")
        
        # Защитник скрывает реальный путь и завышает ценность Honeypot
        perceived_payoffs = self.real_payoffs.copy()
        
        if attacker_type == "low_skill":
            # Новичок верит маскировке: не видит реальную DB и думает, что Honeypot ценен
            visible_edges = [('Start', 'Web'), ('Web', 'Honeypot')]
            perceived_payoffs['Honeypot'] = 120  # Обман сработал: ложная цель кажется сверхценной
        
        elif attacker_type == "high_skill":
            # Профи частично распознает обман благодаря разведке (сканированию)
            visible_edges = [('Start', 'Web'), ('Web', 'DB_Real'), ('Web', 'Honeypot')]
            perceived_payoffs['Honeypot'] = 10   # Распознал ловушку, снизил приоритет
            
        else:
            raise ValueError("Неизвестный тип атакующего")

        # Строим индивидуальный "видимый" граф для текущего злоумышленника
        G_perceived = nx.DiGraph()
        G_perceived.add_edges_from(visible_edges)

        best_path = None
        max_utility = -float('inf')

        # Выбираем целевые узлы, которые видит атакующий
        targets = [node for node in perceived_payoffs if node in G_perceived.nodes and node not in ['Start', 'Web']]
        
        for target in targets:
            if nx.has_path(G_perceived, 'Start', target):
                path = nx.shortest_path(G_perceived, 'Start', target)
                
                # Считаем затраты на путь
                path_cost = 0
                for i in range(len(path) - 1):
                    path_cost += self.edge_costs.get((path[i], path[i+1]), 0)
                
                # Выгода = Воспринимаемая ценность - Затраты
                utility = perceived_payoffs[target] - path_cost
                
                if utility > max_utility:
                    max_utility = utility
                    best_path = path

        # Фиксация реального исхода для защитника
        final_node = best_path[-1] if best_path else None
        actual_damage = self.real_payoffs.get(final_node, 0)
        
        print(f"Выбранный путь атаки: {' -> '.join(best_path)}")
        print(f"Ожидаемая выгода злоумышленника: {max_utility}")
        print(f"Реальный ущерб для системы защитника: {actual_damage} (Успех децепции: {actual_damage == 0})")
        return actual_damage

# Запуск симуляции игры Стекльберга
if __name__ == "__main__":
    game = DeceptionGame()
    
    # Исход 1: Атакующий-Новичок попадает в ловушку
    game.simulate_attacker("low_skill")
    
    # Исход 2: Атакующий-Профи обходит обман
    game.simulate_attacker("high_skill")
```
## 📈 4. Ключевые выводы проекта

1. **Эффективность децепции:** Против низкоквалифицированных нарушителей (`low_skill`) обманные технологии (кибер-децепция) снижают ущерб до нуля, успешно уводя их на Honeypots и заставляя тратить ресурсы впустую.
2. **Ограничения:** Против продвинутых хакеров (`high_skill`) одного обмана мало. Необходимы гибридные стратегии: сочетание децепции с традиционными методами защиты (распределение файрволов, патчинг уязвимостей, классические игры Стекльберга).
3. **Научная ценность:** Данный подход переводит кибербезопасность из реактивного состояния («отражаем атаки по факту») в проактивное («управляем поведением и приоритетами атакующего через контролируемую дезинформацию»).

---
*Отказ от ответственности: Данный аналитический разбор и программный код подготовлены на основе открытых материалов статьи «Harnessing the Power of Deception in Attack Graph-Based Security Games» (GameSec 2020) исключительно в образовательных целях.*

