# Разработка iOS-приложения для планирования задач с использованием SwiftUI

## Содержание
1. [Введение](#введение)
2. [Технический стек](#технический-стек)
3. [Архитектура приложения](#архитектура-приложения)
4. [Модели данных](#модели-данных)
5. [Пользовательский интерфейс](#пользовательский-интерфейс)
6. [Функциональность](#функциональность)
7. [Заключение](#заключение)
8. [Приложения](#приложения)

## Введение

В современном мире эффективное управление задачами становится все более важным аспектом повседневной жизни. Разработанное iOS-приложение представляет собой современное решение для планирования и организации задач, построенное с использованием фреймворка SwiftUI.

### Цели проекта
- Создание интуитивно понятного интерфейса для управления задачами
- Реализация удобной системы категоризации задач
- Обеспечение гибкости в настройке приоритетов и сроков
- Внедрение современных практик разработки iOS-приложений

### Актуальность
В условиях современного ритма жизни и большого количества задач, которые необходимо выполнять ежедневно, эффективное планирование становится критически важным. Разработанное приложение помогает пользователям:
- Организовать свои задачи
- Расставить приоритеты
- Отслеживать прогресс
- Управлять временем

## Технический стек

### Основные технологии
- Swift 5.0+
- SwiftUI
- Combine
- UserDefaults для хранения данных

### Требования к системе
- iOS 15.0+
- Xcode 13.0+
- iPhone/iPad с поддержкой iOS 15.0

### Выбор технологий
SwiftUI был выбран как основной фреймворк для разработки пользовательского интерфейса по следующим причинам:
- Декларативный подход к созданию UI
- Встроенная поддержка анимаций
- Простота интеграции с другими компонентами iOS
- Активная поддержка со стороны Apple

## Архитектура приложения

Приложение построено с использованием архитектуры MVVM (Model-View-ViewModel), что обеспечивает:
- Четкое разделение ответственности
- Удобное тестирование
- Простоту поддержки и расширения

### Структура проекта
```
Planner/
├── Models/
│   ├── Task.swift
│   └── Category.swift
├── Views/
│   ├── TaskListView.swift
│   ├── TaskDetailView.swift
│   ├── CategoryView.swift
│   └── Components/
│       ├── TaskRow.swift
│       └── CategoryRow.swift
├── ViewModels/
│   └── TaskStore.swift
└── Utils/
    └── ColorTheme.swift
```

### Компоненты архитектуры

#### Models
Модели данных представляют собой структуры, описывающие основные сущности приложения:
- Task - задача
- Category - категория

#### Views
Представления отвечают за отображение данных пользователю:
- TaskListView - список задач
- TaskDetailView - детали задачи
- CategoryView - управление категориями

#### ViewModels
ViewModels обеспечивают связь между моделями и представлениями:
- TaskStore - управление задачами и категориями

## Модели данных

### Модель Task
```swift
struct Task: Identifiable, Codable {
    var id = UUID()
    var title: String
    var description: String
    var dueDate: Date
    var priority: Priority
    var category: Category
    var isCompleted: Bool
    
    enum Priority: String, Codable, CaseIterable {
        case low = "Низкий"
        case medium = "Средний"
        case high = "Высокий"
    }
}
```

### Модель Category
```swift
struct Category: Identifiable, Codable {
    var id = UUID()
    var name: String
    var color: String
}
```

### Особенности реализации моделей
- Использование протокола `Identifiable` для уникальной идентификации
- Реализация `Codable` для сериализации/десериализации
- Использование перечислений для ограничения возможных значений

## Пользовательский интерфейс

### Главный экран
Главный экран приложения представляет собой список задач с возможностью:
- Просмотра всех задач
- Фильтрации по категориям
- Сортировки по приоритету и сроку
- Быстрого добавления новых задач

[Здесь нужно вставить скриншот главного экрана приложения]

### Реализация TaskListView
```swift
struct TaskListView: View {
    @StateObject private var taskStore = TaskStore()
    @State private var showingAddTask = false
    
    var body: some View {
        NavigationView {
            List {
                ForEach(taskStore.tasks) { task in
                    TaskRow(task: task)
                }
            }
            .navigationTitle("Задачи")
            .toolbar {
                Button(action: { showingAddTask = true }) {
                    Image(systemName: "plus")
                }
            }
            .sheet(isPresented: $showingAddTask) {
                TaskDetailView(task: nil)
            }
        }
    }
}
```

### Экран создания/редактирования задачи
Форма создания задачи включает:
- Поле для названия
- Поле для описания
- Выбор даты выполнения
- Выбор приоритета
- Выбор категории

[Здесь нужно вставить скриншот экрана создания задачи]

### Реализация TaskDetailView
```swift
struct TaskDetailView: View {
    @Environment(\.dismiss) var dismiss
    @StateObject private var taskStore = TaskStore()
    
    @State private var title: String = ""
    @State private var description: String = ""
    @State private var dueDate = Date()
    @State private var priority: Task.Priority = .medium
    @State private var selectedCategory: Category?
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("Основная информация")) {
                    TextField("Название", text: $title)
                    TextEditor(text: $description)
                        .frame(height: 100)
                }
                
                Section(header: Text("Детали")) {
                    DatePicker("Срок", selection: $dueDate, displayedComponents: .date)
                    Picker("Приоритет", selection: $priority) {
                        ForEach(Task.Priority.allCases, id: \.self) { priority in
                            Text(priority.rawValue).tag(priority)
                        }
                    }
                }
                
                Section(header: Text("Категория")) {
                    Picker("Выберите категорию", selection: $selectedCategory) {
                        ForEach(taskStore.categories) { category in
                            Text(category.name).tag(category as Category?)
                        }
                    }
                }
            }
            .navigationTitle("Новая задача")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Сохранить") {
                        saveTask()
                        dismiss()
                    }
                }
            }
        }
    }
    
    private func saveTask() {
        guard let category = selectedCategory else { return }
        let task = Task(
            title: title,
            description: description,
            dueDate: dueDate,
            priority: priority,
            category: category,
            isCompleted: false
        )
        taskStore.addTask(task)
    }
}
```

### Экран категорий
Управление категориями задач с возможностью:
- Просмотра всех категорий
- Добавления новых категорий
- Редактирования существующих категорий
- Удаления категорий

[Здесь нужно вставить скриншот экрана категорий]

## Функциональность

### Управление задачами
```swift
class TaskStore: ObservableObject {
    @Published var tasks: [Task] = []
    
    func addTask(_ task: Task) {
        tasks.append(task)
        saveTasks()
    }
    
    func updateTask(_ task: Task) {
        if let index = tasks.firstIndex(where: { $0.id == task.id }) {
            tasks[index] = task
            saveTasks()
        }
    }
    
    func deleteTask(_ task: Task) {
        tasks.removeAll { $0.id == task.id }
        saveTasks()
    }
}
```

### Хранение данных
```swift
extension TaskStore {
    private func saveTasks() {
        if let encoded = try? JSONEncoder().encode(tasks) {
            UserDefaults.standard.set(encoded, forKey: "tasks")
        }
    }
    
    private func loadTasks() {
        if let data = UserDefaults.standard.data(forKey: "tasks"),
           let decoded = try? JSONDecoder().decode([Task].self, from: data) {
            tasks = decoded
        }
    }
}
```

### Управление категориями
```swift
extension TaskStore {
    @Published var categories: [Category] = []
    
    func addCategory(_ category: Category) {
        categories.append(category)
        saveCategories()
    }
    
    func updateCategory(_ category: Category) {
        if let index = categories.firstIndex(where: { $0.id == category.id }) {
            categories[index] = category
            saveCategories()
        }
    }
}
```

### Фильтрация и сортировка
```swift
extension TaskStore {
    func filteredTasks(by category: Category?) -> [Task] {
        guard let category = category else { return tasks }
        return tasks.filter { $0.category.id == category.id }
    }
    
    func sortedTasks(by sortOption: SortOption) -> [Task] {
        switch sortOption {
        case .priority:
            return tasks.sorted { $0.priority.rawValue > $1.priority.rawValue }
        case .dueDate:
            return tasks.sorted { $0.dueDate < $1.dueDate }
        case .title:
            return tasks.sorted { $0.title < $1.title }
        }
    }
    
    enum SortOption {
        case priority
        case dueDate
        case title
    }
}
```

## Заключение

Разработанное приложение представляет собой современное решение для управления задачами, которое:
- Обеспечивает удобный пользовательский интерфейс
- Предоставляет гибкие возможности настройки
- Использует современные технологии разработки
- Следует лучшим практикам iOS-разработки

### Достигнутые результаты
1. Создан полнофункциональный планировщик задач
2. Реализована система категоризации
3. Внедрена система приоритетов
4. Обеспечено удобное управление задачами

### Перспективы развития
1. Добавление поддержки уведомлений
2. Интеграция с календарем
3. Добавление статистики выполнения задач
4. Реализация синхронизации с облачными сервисами

## Приложения

### Приложение А: Руководство пользователя

#### Основные функции
1. Создание задачи
   - Нажмите кнопку "+" на главном экране
   - Заполните необходимые поля
   - Нажмите "Сохранить"

2. Редактирование задачи
   - Нажмите на задачу в списке
   - Внесите необходимые изменения
   - Нажмите "Сохранить"

3. Удаление задачи
   - Нажмите и удерживайте задачу
   - Выберите "Удалить" в контекстном меню

4. Управление категориями
   - Перейдите на вкладку "Категории"
   - Используйте кнопку "+" для создания новой категории
   - Нажмите на категорию для редактирования

### Приложение Б: Техническая документация

#### API документация

##### TaskStore
```swift
class TaskStore: ObservableObject {
    // Добавление задачи
    func addTask(_ task: Task)
    
    // Обновление задачи
    func updateTask(_ task: Task)
    
    // Удаление задачи
    func deleteTask(_ task: Task)
    
    // Фильтрация задач
    func filteredTasks(by category: Category?) -> [Task]
    
    // Сортировка задач
    func sortedTasks(by sortOption: SortOption) -> [Task]
}
```

##### Модели данных
```swift
// Задача
struct Task: Identifiable, Codable {
    var id: UUID
    var title: String
    var description: String
    var dueDate: Date
    var priority: Priority
    var category: Category
    var isCompleted: Bool
}

// Категория
struct Category: Identifiable, Codable {
    var id: UUID
    var name: String
    var color: String
}
```

### Приложение В: Тестовые сценарии

#### Модульные тесты
```swift
class TaskStoreTests: XCTestCase {
    var taskStore: TaskStore!
    
    override func setUp() {
        super.setUp()
        taskStore = TaskStore()
    }
    
    func testAddTask() {
        let task = Task(title: "Test", description: "Test", dueDate: Date(), priority: .medium, category: Category(name: "Test", color: "red"), isCompleted: false)
        taskStore.addTask(task)
        XCTAssertEqual(taskStore.tasks.count, 1)
    }
    
    func testDeleteTask() {
        let task = Task(title: "Test", description: "Test", dueDate: Date(), priority: .medium, category: Category(name: "Test", color: "red"), isCompleted: false)
        taskStore.addTask(task)
        taskStore.deleteTask(task)
        XCTAssertEqual(taskStore.tasks.count, 0)
    }
}
```

#### UI тесты
```swift
class PlannerUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        app = XCUIApplication()
        app.launch()
    }
    
    func testAddTask() {
        app.buttons["+"].tap()
        app.textFields["Название"].typeText("Test Task")
        app.buttons["Сохранить"].tap()
        XCTAssertTrue(app.staticTexts["Test Task"].exists)
    }
}
```

### Дополнительные материалы

1. **Скриншоты интерфейса**
   - Главный экран
   - Экран создания задачи
   - Экран категорий
   - Экран настроек

2. **Диаграммы**
   - Архитектурная диаграмма
   - Диаграмма классов
   - Диаграмма последовательности для основных операций

3. **Технические спецификации**
   - Требования к системе
   - API документация
   - Описание алгоритмов

4. **Тестовые сценарии**
   - Модульные тесты
   - Интеграционные тесты
   - UI тесты 