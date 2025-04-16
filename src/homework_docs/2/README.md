# Лекция 2: Углубленное изучение хуков в React

## Содержание лекции

1. 🧠 **Детальный разбор useEffect**
   - Жизненный цикл useEffect
   - Функция очистки и её важность
   - useEffect vs useLayoutEffect
   - Оптимизация и лучшие практики

2. 🛠️ **Обзор других важных хуков**
   - useRef - доступ к DOM и персистентное значение
   - useReducer - управление сложным состоянием
   - useMemo и useCallback - оптимизация производительности
   - useContext - глобальное состояние без prop drilling

3. ⚠️ **Типичные ошибки и антипаттерны**
   - Нарушение правил хуков (условный вызов)
   - Проблемы с зависимостями в useEffect
   - Устаревшие замыкания (Stale Closures)
   - Преждевременная оптимизация

4. 🔌 **Создание пользовательского хука useThemeColor**
   - Структура хука и ThemeProvider
   - Интеграция с проектом
   - Практическое применение

5. 💻 **Практические демонстрации**
   - Жизненный цикл useEffect
   - Использование useRef на практике
   - Управление состоянием с useReducer
   - Оптимизация с useMemo и useCallback

---

## useEffect детально

### Структура и варианты использования

```jsx
useEffect(() => {
  // Код эффекта
  console.log('Компонент обновлен');
  
  // Функция очистки (опционально)
  return () => {
    console.log('Компонент будет обновлен или размонтирован');
  };
}, [/* зависимости */]);
```

| Зависимости | Когда выполняется эффект | Когда выполняется очистка |
|-------------|--------------------------|---------------------------|
| Не указаны  | После каждого рендера    | Перед каждым новым выполнением эффекта |
| `[]`        | Только при монтировании  | Только при размонтировании |
| `[value]`   | При изменении value      | Перед новым выполнением при изменении value |

---

## useRef - 3 основных применения

1. **Доступ к DOM-элементам**

   ```jsx
   const inputRef = useRef(null);
   // ...
   <input ref={inputRef} />
   ```

2. **Сохранение значений между рендерами**

   ```jsx
   const renderCount = useRef(0);
   // Не вызывает ререндер при изменении
   renderCount.current += 1;
   ```

3. **Хранение предыдущего состояния**

   ```jsx
   const prevCount = useRef();
   useEffect(() => {
     prevCount.current = count;
   }, [count]);
   ```

---

## useReducer vs useState

| useState | useReducer |
|----------|------------|
| Простые независимые значения | Сложное взаимосвязанное состояние |
| Мало логики обновления | Сложная логика обновления |
| 1-2 обновления за раз | Множественные обновления за действие |
| ❌ Сложно тестировать | ✅ Легко тестировать (pure functions) |
| ❌ Повторное использование логики | ✅ Централизация логики |

```jsx
const [state, dispatch] = useReducer(reducer, initialState);

// Вызов действия
dispatch({ type: 'increment' });
```

---

## useMemo & useCallback

### useMemo - Мемоизация вычислений

```jsx
// Пересчитывается только при изменении numbers
const sortedItems = useMemo(() => {
  console.log('Sorting items...');
  return [...items].sort();
}, [items]);
```

### useCallback - Мемоизация функций

```jsx
// Создается только при изменении id
const handleClick = useCallback(() => {
  console.log(`Item ${id} clicked`);
}, [id]);
```

**Когда использовать:**

- Для тяжелых вычислений (useMemo)
- Для передачи функций в мемоизированные дочерние компоненты (useCallback)
- Для стабилизации ссылок в зависимостях других хуков

---

## Распространенные ошибки

### 1. Условные вызовы хуков ❌

```jsx
// Нарушение правил хуков!
if (condition) {
  useEffect(() => {/*...*/}, []);
}
```

### 2. Бесконечные циклы в useEffect ❌

```jsx
useEffect(() => {
  setCount(count + 1); // Вызывает ререндер, который запустит эффект снова
}, [count]);
```

### 3. Устаревшие замыкания ❌

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    // Всегда использует начальное значение count
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(timer);
}, []); // Missing dependency: 'count'
```

---

## Правильные решения

### Условная логика внутри эффекта ✅

```jsx
useEffect(() => {
  if (!condition) return;
  // Код эффекта
}, [condition]);
```

### Функциональные обновления ✅

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    setCount(prevCount => prevCount + 1); // Использует актуальное значение
  }, 1000);
  return () => clearInterval(timer);
}, []); // Нет зависимости от count
```

### Стабилизация ссылок ✅

```jsx
// Стабильная ссылка с помощью useMemo
const options = useMemo(() => {
  return { headers: { Authorization: `Bearer ${token}` } };
}, [token]);

useEffect(() => {
  fetchData(options);
}, [options]); // Выполнится только при изменении token
```

---

## Пользовательский хук useThemeColor

```jsx
function useThemeColor() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useThemeColor должен использоваться внутри ThemeProvider');
  }
  return context;
}

// Использование
function ThemedButton() {
  const { colors, toggleTheme } = useThemeColor();
  
  return (
    <button 
      style={{ 
        backgroundColor: colors.background, 
        color: colors.text 
      }}
      onClick={toggleTheme}
    >
      Переключить тему
    </button>
  );
}
```

---

## Ключевые выводы

1. **useEffect требует внимания к деталям**
   - Правильно указывайте зависимости
   - Не забывайте про функции очистки
   - Разделяйте эффекты по их назначению

2. **Выбирайте правильный хук для задачи**
   - useState для простых независимых состояний
   - useReducer для сложной взаимосвязанной логики
   - useRef для персистентных значений без ререндера
   - useMemo/useCallback для оптимизации производительности

3. **Создавайте пользовательские хуки**
   - Инкапсуляция и повторное использование логики
   - Улучшение читаемости компонентов
   - Упрощение тестирования

---

## Рекомендуемые ресурсы

- 📚 [React документация по хукам](https://react.dev/reference/react/hooks)
- 📝 [Полное руководство по useEffect (Dan Abramov)](https://overreacted.io/a-complete-guide-to-useeffect/)
- 🧠 [Мыслим реактивно в React](https://tkdodo.eu/blog/thinking-reactively-in-react)
- 🔍 [useEffect vs useLayoutEffect (Josh Comeau)](https://www.joshwcomeau.com/react/useeffect-vs-uselayouteffect/)
- 🛠️ [Рецепты пользовательских хуков](https://usehooks.com/)
