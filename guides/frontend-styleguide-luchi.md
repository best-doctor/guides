# Frontend Styleguide LUCHI

> Этот документ - primary source of truth для frontend-стандартов ЛУЧИ.
>
> Связанный skill: [frontend-styleguide-luchi](https://bestdoctor.gitlab.yandexcloud.net/bestdoctor/common/skills/-/blob/main/frontend-styleguide-luchi/SKILL.md?ref_type=heads)
>
> Если skill неудобен, неполон или устарел, меняем этот гайд или процесс генерации skill, но не редактируем производный skill вручную.

## Базовые принципы

- Единообразие важнее локальных предпочтений.
- Понятность важнее краткости, cleverness, микрооптимизаций и комментариев.
- Максимум проверок выносим в инструменты: TypeScript, ESLint, Prettier, tests.
- Имена отражают бизнес-смысл, а не визуальную реализацию.
- Пользователь всегда получает feedback на `loading`, `empty`, `error`, `success`.

## Tooling

### Linting

- [`eslint-config-bestdoctor`](https://github.com/best-doctor/eslint-config-bestdoctor) считаем legacy-базой.
- Для новых проектов нужен новый, более простой и достаточный конфиг.
- `TODO`: разработать, согласовать и внедрить новый базовый ESLint-конфиг для новых frontend-проектов.
- Проектные правила и плагины допустимы, если они не ломают единый стиль.
- Легаси-нарушения временно переводим в `warn`; по мере рефакторинга исключения убираем.
- Отключение правила линтинга:
    - только точечно;
    - только с объяснением причины;
    - по возможности со ссылкой на задачу.

### Formatting

```json
{
    "printWidth": 120,
    "semi": true,
    "singleQuote": true,
    "tabWidth": 2,
    "trailingComma": "es5",
    "bracketSpacing": true,
    "arrowParens": "always",
    "endOfLine": "lf"
}
```

- Конфиг задается в корневом `.prettierrc`.
- Если проектный конфиг отсутствует, создаем его в этом виде.
- Используем `eslint-config-prettier`, чтобы не конфликтовать с ESLint-конфигом.
- Локальные отклонения от этого конфига допустимы только по явной проектной причине.

### Styling

- Следуем существующему проектному стеку.
- По умолчанию предпочитаем `Tailwind CSS` или `CSS Modules`.
- Не добавляем новый styling-подход без явной причины.

## Архитектура

Используем FSD-lite.

### Слои

- `src/app` - providers, app-level config, global styles.
- `src/pages` - route-level composition.
- `src/entities` - бизнес-сущности; основной слой предметной логики и UI вокруг сущностей.
- `src/shared` - переиспользуемые UI-компоненты, общие utils, constants, types и инфраструктурные модули.
- `src/widgets` и `src/features` - опциональны; добавляются только если проект реально выигрывает от отдельного orchestration-слоя.

### Структура slice

Внутри slice используем директории по назначению:

- `ui` - view-слой.
- `hooks` - hooks и slice-level logic.
- `stores` - локальные и slice-level stores.
- `utils` - локальные утилиты.
- `constants` - константы slice.
- `types` - типы slice.

`pages` внутри `entities` не создаем. Глобальные singleton-like вещи размещаем в `app` или `shared`, а не в отдельном top-level `services`.

### Границы импортов

- Верхний слой может импортировать нижний.
- Импорты между slice одного слоя запрещены.
- Циклические зависимости запрещены.
- `index.ts`-реэкспорты по умолчанию запрещены.
- Barrel-файлы допустимы только как осознанный public API, если это не ухудшает tree-shaking, bundle size и поиск мертвого кода.

## Данные и стейт

### Backend и запросы

Используем `@tanstack/react-query`.

Перед тем как писать ручной клиент или query factory:

1. Проверяем, нет ли уже автогенеренных клиентов, hooks или готового инфраструктурного слоя для работы с backend.
2. Проверяем project-level инструкции (`AGENTS.md`, `.cursorrules`, `README`).
3. Только если готового решения нет, пишем ручную реализацию.

### Query keys

Для ручной реализации используем key factory:

```ts
export function getRequestsKeys<TListParams, TItemKey = string>(
    entityName: string,
) {
    return {
        all: [entityName] as const,
        lists: [entityName, "list"] as const,
        list: (params: TListParams) => [entityName, "list", params] as const,
        items: [entityName, "item"] as const,
        item: (key: TItemKey) => [entityName, "item", key] as const,
    };
}
```

- `all` - все запросы сущности.
- `lists` - все списки.
- `list(params)` - конкретный список.
- `items` - все item-запросы.
- `item(key)` - конкретная запись.

### Query hooks

- Используем object API: `useQuery({ ... })`, `useInfiniteQuery({ ... })`, `useMutation({ ... })`.
- Хуки должны принимать `options`, если это не делает API хуже.
- `staleTime` и invalidation определяем по характеру данных, а не по привычке.
- На `mutation` по умолчанию инвалидируем релевантные `lists` и `item(id)`.

### Нейминг query/mutation hooks

- Одна запись: `useUserQuery`
- Специфический запрос: `useUserByEmailQuery`
- Список: `useUsersListQuery`
- Бесконечный список: `useInfiniteUsersListQuery`
- Бесконечный поиск: `useInfiniteSearchUsersQuery`
- Создание: `useCreateUserMutation`
- Обновление записи: `useUpdateUserMutation`
- Обновление поля: `useUpdateUserNameMutation`
- Удаление: `useRemoveUserMutation`

### Формы и локальный стейт

- Для форм используем `react-hook-form`.
- Все, что может быть локальным, оставляем локальным.
- Глобальный стейт - исключение, а не норма.
- Глобальный store используем только для реально shared-состояния: авторизация, глобальный виджет, cross-page state.

### Loading / Empty / Error

- Предпочитаем `Suspense + ErrorBoundary + Skeletons`.
- Не оставляем пользователя на пустом экране.
- Если данных нет - показываем empty state.
- Если запрос упал - показываем понятный error state и следующий шаг.

## Нейминг

### Общие правила

- Используем английский язык.
- Название должно быть семантичным и бизнес-ориентированным.
- Понятность важнее длины и "красоты".
- Называем сущность по роли, а не по иконке, цвету или layout.

Хорошо: `TogglePasswordVisibilityButton`

Плохо: `EyeIconButton`

### Функции

- Имя функции начинается с глагола.
- `get*` - возвращает значение.
- `validate*`, `check*` - возвращает `boolean`.
- `format*` - преобразует данные.
- `set*` - пишет в state/store/ref.
- `handle*` - внутренний handler.
- `on*` - callback prop компонента.

### Переменные

- Переменная - существительное.
- Boolean-флаги: `is*`, `has*`, `can*`, `should*`, `was*`, `did*`.
- Коллекции называем во множественном числе: `users`, `errors`, `phoneNumbers`.
- Количество сущностей - `*Number`.
- Порядковый номер - `*Index`.
- Raw/form payload - `*Data`.
- Состояние называем как состояние: `isOpen`, `isEnabled`; не смешиваем state и action.

### Компоненты, файлы, директории

- Компоненты - `PascalCase`.
- Файлы и директории - `camelCase`.
- Имя файла по возможности совпадает с главным export.

### Naming reference

- Хорошее имя описывает роль в бизнес-логике, а не реализацию.
- `usersPhoneNumbers` - номера телефонов пользователей.
- `userPhoneNumbers` - номера телефонов одного пользователя.
- `usersPhoneNumber` - плохое имя: число сущностей и число коллекции конфликтуют.
- Для UI-компонентов имя строится от назначения: `SubmitOrderButton`, `UserProfileCard`, `TogglePasswordVisibilityButton`.
- Имена вроде `BlueCard`, `LeftPanel`, `EyeIconButton` допустимы только если цвет, позиция или иконка и есть бизнес-смысл сущности, что почти никогда не так.

## Дизайн кода

### Single responsibility

Одна функция делает одну задачу и делает ее хорошо.

Признаки, что функцию пора делить:

- в названии есть `And`;
- `boolean`-аргумент переключает поведение;
- функция разрастается на сотни строк;
- внутри смешаны orchestration, data transform и view logic.

### Параметры функций

- `1-2` аргумента - обычно positional.
- `3+` аргумента - объект.
- Один объект тоже допустим, если функция будет расширяться.
- Исключение: thin wrapper над backend/API может принимать много полей, если функция почти не содержит логики.

### Комментарии

- Комментарии по умолчанию не пишем.
- Предпочитаем self-documenting code через нейминг и декомпозицию.
- Допустимы:
    - причина отключения линтинга;
    - `TODO` / tech debt с ссылкой на задачу;
    - краткое объяснение нетривиального решения, которое нельзя сделать очевиднее кодом.

## Reference

### FSD-lite example

```text
src/
  app/        # providers, app config, global styles
  pages/      # route-level composition
  entities/   # User/, Order/, Product/
  shared/     # ui/, utils/, constants/, types/
  widgets/    # optional
  features/   # optional
```

### Forbidden imports

- `entities/user` -> `entities/order` запрещено.
- `shared/ui/button` -> `entities/user` запрещено.
- `pages/userPage` -> `pages/orderPage` обычно запрещено; общую логику выносим ниже.
- Если orchestration нужен между несколькими entities, поднимаем его в `pages`, `widgets` или `features`.

### React Query reference

- Предпочитаем generic-обертки с `options`, чтобы хук можно было переиспользовать без копипаста.
- Для списков используем `list(params)`, для одной сущности `item(id)`.
- `invalidateQueries` бьет либо по конкретному `item`, либо по набору `lists`; не инвалидируем `all`, если можно сузить область.
- Если проект использует автогенерацию API-клиента и hooks, придерживаемся project-native pattern, а не насаждаем manual factory поверх него.

## Canonical examples

### Query key factory

```ts
export function getRequestsKeys<TListParams, TItemKey = string>(
    entityName: string,
) {
    return {
        all: [entityName] as const,
        lists: [entityName, "list"] as const,
        list: (params: TListParams) => [entityName, "list", params] as const,
        items: [entityName, "item"] as const,
        item: (key: TItemKey) => [entityName, "item", key] as const,
    };
}
```

### Query hook

```ts
import { useQuery, type UseQueryOptions } from "@tanstack/react-query";

type FetchError = { message: string };
type UserResponse = { id: string; name: string; email: string };
type UserResult = UserResponse;

const userKeys = getRequestsKeys<never, string>("users");

async function getUser(id: string): Promise<UserResponse> {
    const response = await fetch(`/api/users/${id}`);

    if (!response.ok) {
        throw new Error("Network response was not ok");
    }

    return response.json();
}

type UseUserQueryOptions<TResult> = Partial<
    UseQueryOptions<UserResponse, FetchError, TResult>
>;

export function useUserQuery<TResult = UserResult>(
    id: string,
    options?: UseUserQueryOptions<TResult>,
) {
    return useQuery<UserResponse, FetchError, TResult>({
        queryKey: userKeys.item(id),
        queryFn: () => getUser(id),
        staleTime: 60_000,
        ...options,
    });
}
```

### Mutation hook

```ts
import {
    useMutation,
    useQueryClient,
    type UseMutationOptions,
} from "@tanstack/react-query";

type FetchError = { message: string };
type UserResponse = { id: string; name: string };
type UpdateUserParams = { id: string; name: string };

async function updateUser(params: UpdateUserParams): Promise<UserResponse> {
    const response = await fetch(`/api/users/${params.id}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(params),
    });

    if (!response.ok) {
        throw new Error("Network response was not ok");
    }

    return response.json();
}

export function useUpdateUserMutation(
    options?: UseMutationOptions<UserResponse, FetchError, UpdateUserParams>,
) {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: updateUser,
        onSuccess: (data) => {
            queryClient.invalidateQueries({ queryKey: userKeys.lists });
            queryClient.invalidateQueries({ queryKey: userKeys.item(data.id) });
        },
        ...options,
    });
}
```

### Suspense boundary

```tsx
import { Suspense } from "react";
import { ErrorBoundary } from "react-error-boundary";

export function UserPage({ userId }: { userId: string }) {
    return (
        <ErrorBoundary fallback={<div>Something went wrong</div>}>
            <Suspense fallback={<div>Loading...</div>}>
                <UserProfile userId={userId} />
            </Suspense>
        </ErrorBoundary>
    );
}
```

## Требования к derived skill

- Derived skill создается из этого guide, а не дописывается вручную.
- Guide должен содержать не только policy, но и critical reference/examples, которые нужны для корректной генерации кода агентом.
- Progressive guide должен содержать не только policy, но и нормативную базу для производных материалов.
- Derived skill по умолчанию строится как progressive disclosure:
    - `SKILL.md` содержит только quick start, critical defaults и ссылки;
    - подробности выносятся в отдельные derived files вроде `reference.md`, `examples.md`, `config.md`;
    - ссылки остаются one-level deep.
- Skill берет из guide:
    - core rules;
    - architecture constraints;
    - naming rules;
    - React Query / state rules;
    - canonical examples, если они нужны агенту для качественной генерации.

## Короткий checklist

- Архитектура укладывается в FSD-lite.
- Импорты соблюдают границы слоев.
- Query/mutation hooks названы консистентно.
- Loading, empty и error states явные.
- Названия отражают бизнес-смысл.
- Линт и форматирование не подавлены без причины.
