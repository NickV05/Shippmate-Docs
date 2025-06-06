# Руководство по стилю интерфейса ShippMate (RU)

## 1. Палитра цветов

| Назначение | Hex | Пример |
|------------|-----|--------|
| **Основной** (бренд / основные CTA) | `#6F42C1` | <span style="background:#6F42C1;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Акцент** (ховер, выделения) | `rgb(255, 199, 74)` | <span style="background:rgb(255,199,74);width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Вторичный** (границы, линии таблиц) | `#E1E7ED` | <span style="background:#E1E7ED;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Текст** | `#2C3E50` | <span style="background:#2C3E50;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Светлый фон** | `#F8F9FA` | <span style="background:#F8F9FA;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Успех** | `#5CB85C` | <span style="background:#5CB85C;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Предупреждение** | `#F0AD4E` | <span style="background:#F0AD4E;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Опасность** | `#D9534F` | <span style="background:#D9534F;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |

### Рекомендации по использованию
* Основной — для бренд-элементов, навигации и основных кнопок.
* Акцент — для состояний _hover_ / активных элементов.
* Вторичный — для тонких линий, отключённых контролов.
* Больше белого / **светлого фона** для лёгкого, воздушного интерфейса.

---

## 2. Типографика

```
-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif
```

| Элемент | Размер | Насыщенность |
|---------|--------|--------------|
| H1 | 24 px | 600 |
| H2 | 20 px | 600 |
| Текст | 14–16 px | 400 |

Цвет основного текста — `#2C3E50`; для второстепенного текста используйте 70 % непрозрачности.

---

## 3. Обрамление компонентов

* **Границы:** `1px solid #E1E7ED`  
* **Скругление:** `8px` у карточек, инпутов, таблиц, модальных окон  
* **Тень:** `0 1px 3px rgba(0,0,0,0.05)` (лёгкое возвышение)

### Кнопки
| Тип | Обычное состояние | Наведение / фокус |
|-----|-------------------|-------------------|
| Primary | фон `#335C81`, текст `#FFF` | фон `#3E8989` |
| Secondary | фон `#FFF`, граница `#335C81`, текст `#335C81` | фон `#E1E7ED` |

### Таблицы
* Заголовок: фон `#F8F9FA`, текст `#2C3E50`
* При наведении строка: фон `#F8F9FA`

### Карточки и модальные окна
* Заголовок: фон `#F8F9FA`, цвет заголовка `#335C81`  
* Подвал: верхняя граница `#E1E7ED`

---

## 4. Логотипы

| Контекст | URL ассета |
|----------|-----------|
| Светлый фон (по умолчанию) | `https://res.cloudinary.com/dyto7dlgt/image/upload/v1740961280/PurpleOnWhite-removebg-preview_erpuod.png` |
| Тёмный / цветной фон | `https://res.cloudinary.com/dyto7dlgt/image/upload/v1732386315/logo_white_back_wx4051.png` |

Рекомендованные размеры отображения:
* Навигация (десктоп): **160 × auto**
* Мобильное меню / иконка: **50 × auto**

Не растягивайте изображение, сохраняйте отступы для визуального центра.

---

## 5. Отступы и ритм

* **Отступы секций (десктоп / планшет / мобайл):** 60 / 40 / 30 px сверху и снизу.  
* **Внутренний паддинг карточек:** 12 px.  
* **Боковые отступы страницы:** 16 px десктоп, 12 px мобайл.

---

## 6. Быстрые CSS-переменные

```css
:root {
  --primary-color: #335C81;
  --accent-color: #3E8989;
  --secondary-color: #E1E7ED;
  --text-color: #2C3E50;
  --light-bg: #F8F9FA;
}
```

---

# ShippMate Front-End Style Guide (EN)


## 1. Colour Palette

| Purpose      | Hex | Example |
|--------------|-----|---------|
| **Primary** (brand / primary CTAs) | `#6F42C1` | <span style="background:#6F42C1;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Accent** (hover, highlights) | `rgb(255, 199, 74)` | <span style="background:rgb(255,199,74);width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Secondary** (borders, table lines) | `#E1E7ED` | <span style="background:#E1E7ED;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Text** (default body) | `#2C3E50` | <span style="background:#2C3E50;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Light background** | `#F8F9FA` | <span style="background:#F8F9FA;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Success** | `#5CB85C` | <span style="background:#5CB85C;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Warning** | `#F0AD4E` | <span style="background:#F0AD4E;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |
| **Danger** | `#D9534F` | <span style="background:#D9534F;width:16px;height:16px;display:inline-block;border:1px solid #ccc;"></span> |

### Usage ratios
* Primary for brand elements, nav, solid buttons.
* Accent for hover / active states.
* Secondary for 1-px lines & disabled controls.
* Plenty of white / _light background_ for an airy look.

---

## 2. Typography

```
-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif
```

| Element      | Size | Weight |
|--------------|------|--------|
| H1           | 24 px | 600 |
| H2           | 20 px | 600 |
| Body         | 14-16 px | 400 |

Body text colour: `#2C3E50`. Use 70 % opacity for secondary copy.

---

## 3. Component Chrome

* **Borders:** `1px solid #E1E7ED`  
* **Border radius:** `8px` on cards, inputs, tables, modals.  
* **Shadow:** `0 1px 3px rgba(0,0,0,0.05)` (subtle lift)

### Buttons
| Type      | Normal | Hover / Focus |
|-----------|--------|---------------|
| Primary   | bg `#335C81`, text `#FFF` | bg `#3E8989` |
| Secondary | bg `#FFF`, border `#335C81`, text `#335C81` | bg `#E1E7ED` |

### Tables
* Header bg `#F8F9FA` → text `#2C3E50`
* Row hover bg `#F8F9FA`

### Cards & Modals
* Header bg `#F8F9FA`, title colour `#335C81`  
* Footer border-top `#E1E7ED`

---

## 4. Logos

| Context | Asset URL |
|---------|-----------|
| Light backgrounds (default) | `https://res.cloudinary.com/dyto7dlgt/image/upload/v1740961280/PurpleOnWhite-removebg-preview_erpuod.png` |
| Dark / coloured backgrounds | `https://res.cloudinary.com/dyto7dlgt/image/upload/v1732386315/logo_white_back_wx4051.png` |

Recommended display sizes:
* Desktop nav: **160 × auto**
* Mobile / icon: **50 × auto**

Do **not** stretch; leave sufficient padding so the glyph feels centred.

---

## 5. Spacing & Rhythm

* **Section padding (desktop / tablet / mobile):** 60 / 40 / 30 px top & bottom.  
* **Card internal gap:** 12 px.  
* **Page gutter:** 16 px desktop, 12 px mobile.

---

## 6. Quick CSS helpers

```css
:root {
  --primary-color: #335C81;
  --accent-color: #3E8989;
  --secondary-color: #E1E7ED;
  --text-color: #2C3E50;
  --light-bg: #F8F9FA;
}
```
