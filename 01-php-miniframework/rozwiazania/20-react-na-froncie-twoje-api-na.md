# Rozdział 20: React na froncie, Twoje API na backendzie

## Zadanie 1: modal dodawania zadania

Komponent `Modal` i `TaskItem` zostają bez zmian. Wszystko dzieje się w `TaskList`.

Nowe stany pod `const [error, setError] = useState(null);`:

```jsx
    // Modal dodawania: czy otwarty i błąd pod polem
    const [adding, setAdding] = useState(false);
    const [addError, setAddError] = useState(null);
```

Zamiast dotychczasowego `addTask` dwie funkcje:

```jsx
    function openAdd() {
        setTitle('');
        setAddError(null);
        setAdding(true);
    }

    async function addTask(e) {
        e.preventDefault();
        setError(null);

        const response = await api('POST', '/api/tasks', { title: title });

        // Błąd walidacji zostaje w modalu, pod polem
        if (response.status === 422) {
            const data = await response.json();
            setAddError(data.errors.title);
            return;
        }

        if (!response.ok) {
            setAdding(false);
            await showError(response);
            return;
        }

        const task = await response.json();
        setTasks(current => [task, ...current]);
        setAdding(false);
    }
```

W JSX nagłówek `<h1>` i cały `<form onSubmit={addTask} ...>` nad listą zastępuje:

```jsx
            <div className="flex items-center justify-between mb-6">
                <h1 className="text-2xl font-bold">Zadania</h1>
                <button onClick={openAdd} className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700">
                    Nowe zadanie
                </button>
            </div>
```

A obok dwóch modali z kroku 11 trzeci:

```jsx
            <Modal open={adding} title="Nowe zadanie" onClose={() => setAdding(false)}>
                <form onSubmit={addTask}>
                    <input
                        type="text"
                        value={title}
                        onChange={e => setTitle(e.target.value)}
                        placeholder="Co jest do zrobienia?"
                        className={'w-full border rounded-lg px-3 py-2 ' + (addError ? 'border-red-500' : 'border-gray-300')}
                    />
                    {addError && <p className="mt-1 text-sm text-red-600">{addError}</p>}

                    <div className="flex justify-end gap-2 mt-6">
                        <button type="button" onClick={() => setAdding(false)} className="px-4 py-2 rounded-lg border border-gray-300 hover:bg-gray-50">
                            Anuluj
                        </button>
                        <button type="submit" className="px-4 py-2 rounded-lg bg-blue-600 text-white hover:bg-blue-700">
                            Dodaj
                        </button>
                    </div>
                </form>
            </Modal>
```

**Na co patrzeć przy review:** `Modal` ma zostać nietknięty; jeśli uczeń go zmieniał, zapytaj po co, bo zwykle to objaw tego, że stan otwarcia trzyma w złym miejscu. `addError` jest osobnym stanem obok `error`: błąd walidacji należy do okna, a 401, 419 czy 500 dalej idą na górę strony przez `showError`. Przy 422 okno ma zostać otwarte z wpisanym tekstem; częsty błąd to `setAdding(false)` przed sprawdzeniem statusu, przez co błąd mignie i zniknie razem z oknem. Czyszczenie pola i błędu ma być przy otwieraniu (`openAdd`), nie tylko po sukcesie: inaczej po „Anuluj” albo Escape przy kolejnym otwarciu zostaje stary tekst i stary błąd. `type="button"` przy „Anuluj”, bo inaczej przycisk wyśle formularz. Formularz musi być `<form onSubmit>`, nie sam przycisk z `onClick`, żeby Enter w polu dodawał zadanie. Fokus po otwarciu ląduje w polu sam, przez `showModal()`; `autoFocus` nie jest potrzebny. Poproś o pokazanie: Escape, ponowne otwarcie (pole puste), puste „Dodaj” (422 w oknie, w Network `POST /api/tasks` 422), poprawne dodanie (201, zadanie na górze). Punkt 3 „Sprawdź, czy działa” [rozdziału 20](../../../ebook-dev-start/01-php-miniframework/rozdzialy/20-react-na-froncie-twoje-api-na.md) dotyczy teraz pola w oknie.

## Zadanie 2: filtr bez serwera

W `TaskList` nowy stan i lista wyliczana przed `return`:

```jsx
    const [filter, setFilter] = useState('all');

    const visibleTasks = tasks.filter(task => {
        if (filter === 'open') {
            return !task.done;
        }
        if (filter === 'done') {
            return task.done;
        }
        return true;
    });

    function filterButton(value, label) {
        const active = filter === value;

        return (
            <button
                onClick={() => setFilter(value)}
                className={'px-3 py-1 rounded-lg text-sm ' + (active ? 'bg-blue-600 text-white' : 'bg-white text-gray-700')}
            >
                {label}
            </button>
        );
    }
```

W JSX nad listą, a w `map` zamiast `tasks` lista `visibleTasks`:

```jsx
            <div className="flex items-center gap-2 mt-4">
                {filterButton('all', 'Wszystkie')}
                {filterButton('open', 'Otwarte')}
                {filterButton('done', 'Zrobione')}
            </div>
```

```jsx
                    {visibleTasks.map(task => (
                        <TaskItem key={task.id} task={task} onDone={markDone} onEdit={openEdit} onRemove={setTaskToDelete} />
                    ))}
```

Warunek „Brak zadań.” też ma sprawdzać `visibleTasks.length`, inaczej przy filtrze „Zrobione” bez zrobionych zadań lista będzie po prostu pusta, bez napisu.

**Na co patrzeć przy review:** filtrowana lista ma być wyliczana przy każdym rysowaniu, a nie trzymana w drugim stanie. Jeśli uczeń zrobił `const [visible, setVisible] = useState(...)`, to po dodaniu zadania albo kliknięciu „Zrobione” filtr się rozjedzie, bo trzeba pamiętać o aktualizacji dwóch stanów; pokaż mu to na przykładzie. `tasks` zostaje nietknięte, filtr nie usuwa niczego ze stanu. Po kliknięciu filtra w Network nie może pojawić się żadne żądanie. `done` z bazy to liczba 0 lub 1, więc `return task.done` działa przez „prawdziwość” liczby; zapytaj, co by było, gdyby w JSX napisał `{task.done && ...}` (React wypisze „0”).

## Zadanie 3: licznik i ładowanie

```jsx
    const [loading, setLoading] = useState(true);

    async function loadTasks() {
        const response = await api('GET', '/api/tasks');
        setLoading(false);

        if (!response.ok) {
            await showError(response);
            return;
        }

        setTasks(await response.json());
    }

    const doneCount = tasks.filter(task => task.done).length;
```

W JSX licznik obok filtrów i trzy stany listy:

```jsx
                <span className="ml-auto text-sm text-gray-600">
                    {doneCount} z {tasks.length} zrobionych
                </span>
```

```jsx
            {loading ? (
                <p className="mt-4 text-gray-500">Ładowanie...</p>
            ) : visibleTasks.length === 0 ? (
                <p className="mt-4 text-gray-500">Brak zadań.</p>
            ) : (
                <ul className="mt-4 space-y-2">
                    {visibleTasks.map(task => (
                        <TaskItem key={task.id} task={task} onDone={markDone} onEdit={openEdit} onRemove={setTaskToDelete} />
                    ))}
                </ul>
            )}
```

**Na co patrzeć przy review:** `doneCount` ma być zwykłą stałą liczoną z `tasks`, nie stanem; wtedy licznik sam się zgadza po każdym dodaniu, oznaczeniu i usunięciu. Licznik liczy z `tasks`, a nie z `visibleTasks`, bo przy filtrze „Otwarte” pokazywałby „0 z 3”. `setLoading(false)` musi się wykonać także przy błędzie, inaczej po 401 albo 500 „Ładowanie...” zostanie na zawsze; jeśli uczeń wstawił je tylko po `setTasks`, poproś go, żeby wyłączył serwer PHP i odświeżył stronę. Zagnieżdżony operator `? :` jest czytelny przy trzech przypadkach; przy czwartym lepiej wydzielić osobną funkcję.
