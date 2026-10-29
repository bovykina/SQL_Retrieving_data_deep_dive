## Задание 00 — Поиск пиццерий без посетителей.
Вывести названия и рейтинги пиццерий, которые никто не посещал, используя соединение таблиц без запрещённых конструкций.
```
SELECT
	PI.NAME,
	PI.RATING
FROM
	PIZZERIA PI
	LEFT JOIN PERSON_VISITS PV ON PV.PIZZERIA_ID = PI.ID
WHERE
	PV.ID IS NULL
```

## Задание 01 — Определение пропущенных дат посещений.
Найти даты с 1 по 10 января 2022 года, в которые отсутствовали посещения обоих указанных клиентов.
```
SELECT
	MISSING_DATE::DATE
FROM
	GENERATE_SERIES(
		'2022-01-01'::DATE,
		'2022-01-10'::DATE,
		'1 day'::INTERVAL
	) MISSING_DATE
	LEFT JOIN PERSON_VISITS PV ON PV.VISIT_DATE = MISSING_DATE
	AND (
		PV.PERSON_ID = '1'
		OR PV.PERSON_ID = '2'
	)
WHERE
	PV.ID IS NULL
ORDER BY
	MISSING_DATE
```

## Задание 02 — Полное сопоставление людей и пиццерий.
Получить полный список людей и пиццерий с данными о посещениях за период с 1 по 3 января 2022 года, сохранив записи без совпадений.
```
SELECT
	CASE
		WHEN P.NAME IS NULL THEN '-'
		ELSE P.NAME
	END AS PERSON_NAME,
	VISIT_DATE,
	CASE
		WHEN PI.NAME IS NULL THEN '-'
		ELSE PI.NAME
	END AS PIZZERIA_NAME
FROM
	GENERATE_SERIES(
		'2022-01-01'::DATE,
		'2022-01-03'::DATE,
		'1 day'::INTERVAL
	) VISIT_DATES
	LEFT JOIN PERSON_VISITS PV ON PV.VISIT_DATE = VISIT_DATES
	FULL OUTER JOIN PERSON P ON P.ID = PV.PERSON_ID
	FULL OUTER JOIN PIZZERIA PI ON PI.ID = PV.PIZZERIA_ID
ORDER BY
	PERSON_NAME,
	VISIT_DATE,
	PIZZERIA_NAME
```

## Задание 03 — Выделение генерации дат в CTE.
Переписать запрос поиска пропущенных дат с использованием общего табличного выражения.
```
WITH
	DATES AS (
		SELECT
			GENERATE_SERIES(
				'2022-01-01'::DATE,
				'2022-01-10'::DATE,
				'1 day'::INTERVAL
			) MISSING_DATE
	)
SELECT
	MISSING_DATE::DATE
FROM
	DATES
	LEFT JOIN PERSON_VISITS PV ON PV.VISIT_DATE = MISSING_DATE
	AND (
		PV.PERSON_ID = '1'
		OR PV.PERSON_ID = '2'
	)
WHERE
	PV.ID IS NULL
ORDER BY
	MISSING_DATE
```

## Задание 04 — Поиск пиццы по составу и месту продажи.
Найти пиццы с грибами или пепперони, указав названия пиццерий и цены.
```
SELECT
	M.PIZZA_NAME,
	PI.NAME PIZZERIA_NAME,
	M.PRICE
FROM
	PIZZERIA PI
	RIGHT JOIN MENU M ON PI.ID = M.PIZZERIA_ID
WHERE
	M.PIZZA_NAME = 'mushroom pizza'
	OR M.PIZZA_NAME = 'pepperoni pizza'
ORDER BY
	PIZZA_NAME,
	PIZZERIA_NAME
```

## Задание 05 — Отбор женщин по возрасту.
Найти имена женщин старше 25 лет и отсортировать их по алфавиту.
```
SELECT
	NAME
FROM
	PERSON
WHERE
	AGE > 25
	AND GENDER = 'female'
ORDER BY
	NAME
```

## Задание 06 — Анализ заказов двух клиентов.
Получить названия пицц и пиццерий, в которых заказывали пиццу Денис или Анна.
```
SELECT
	M.PIZZA_NAME,
	PI.NAME PIZZERIA_NAME
FROM
	PERSON_ORDER PO
	JOIN MENU M ON M.ID = PO.MENU_ID
	JOIN PIZZERIA PI ON PI.ID = M.PIZZERIA_ID
	JOIN PERSON P ON P.ID = PO.PERSON_ID
	AND (
		P.NAME = 'Denis'
		OR P.NAME = 'Anna'
	)
ORDER BY
	PIZZA_NAME,
	PIZZERIA_NAME
```

## Задание 07 — Поиск пиццерии с подходящей ценой.
Определить, какую пиццерию посетил Дмитрий 8 января 2022 года, где можно было заказать пиццу дешевле 800 рублей.
```
SELECT DISTINCT
	PI.NAME PIZZERIA_NAME
FROM
	PERSON_VISITS PV
	JOIN PERSON P ON P.ID = PV.PERSON_ID
	AND (
		P.NAME = 'Dmitriy'
		AND PV.VISIT_DATE = '2022-01-08'
	)
	JOIN PIZZERIA PI ON PI.ID = PV.PIZZERIA_ID
	JOIN MENU M ON PI.ID = M.PIZZERIA_ID
	AND (PRICE < '800')
```

## Задание 08 — Поиск клиентов с определёнными предпочтениями.
Найти мужчин из Москвы или Самары, которые заказывали пиццу с грибами, пепперони или оба варианта.
```
SELECT
	NAME
FROM
	PERSON_ORDER PO
	JOIN PERSON P ON P.ID = PO.PERSON_ID
	AND (
		ADDRESS = 'Moscow'
		OR ADDRESS = 'Samara'
	)
	AND GENDER = 'male'
	JOIN MENU M ON M.ID = PO.MENU_ID
	AND (
		PIZZA_NAME = 'mushroom pizza'
		OR PIZZA_NAME = 'pepperoni pizza'
	)
ORDER BY
	NAME DESC
```

## Задание 09 — Выявление общих предпочтений в заказах.
Найти женщин, которые заказывали и сырную, и пепперони-пиццу, независимо от даты и пиццерии.
```
SELECT
	P.NAME
FROM
	PERSON P
	INNER JOIN PERSON_ORDER PO ON P.ID = PO.PERSON_ID
	INNER JOIN MENU M ON M.ID = PO.MENU_ID
WHERE
	M.PIZZA_NAME = 'pepperoni pizza'
	AND P.GENDER = 'female'
INTERSECT
SELECT
	P.NAME
FROM
	PERSON P
	INNER JOIN PERSON_ORDER PO ON P.ID = PO.PERSON_ID
	INNER JOIN MENU M ON M.ID = PO.MENU_ID
WHERE
	M.PIZZA_NAME = 'cheese pizza'
	AND P.GENDER = 'female'
ORDER BY
	1
```

## Задание 10 — Поиск людей с одинаковым адресом.
Найти пары людей, проживающих по одному адресу, и отсортировать результаты по именам и общему адресу.
```
SELECT
	P1.NAME PERSON_NAME1,
	P2.NAME PERSON_NAME2,
	P1.ADDRESS COMMON_ADDRESS
FROM
	PERSON P1
	JOIN PERSON P2 ON P1.ADDRESS = P2.ADDRESS
WHERE
	P1.ID > P2.ID
ORDER BY
	PERSON_NAME1,
	PERSON_NAME2,
	COMMON_ADDRESS
```
