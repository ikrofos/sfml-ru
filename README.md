# Создание собственных объектов с помощью массива вершин

> Вольный перевод документации [Designing your own entities with vertex arrays](https://www.sfml-dev.org/tutorials/2.5/graphics-vertex-array.php).

## Введение

SFML предоставляет простые классы для наиболее распространенных 2D-объектов. И хотя из этих строительных блоков можно легко создать более сложные объекты, это не всегда самое эффективное решение. Например, вы очень быстро достигнете пределов своей видеокарты, если нарисуете большое количество спрайтов. Причина в том, что производительность во многом зависит от количества вызовов функции отрисовки (`draw`). Действительно, каждый вызов включает в себя установку набора состояний OpenGL, сброс матриц, изменение текстур и т. д. Все это требуется даже при простом рисовании двух треугольников (спрайт). Это далеко не оптимально для вашей видеокарты: современные графические процессоры предназначены для обработки больших пакетов треугольников, обычно от нескольких тысяч до миллионов.

Чтобы заполнить этот пробел, SFML предоставляет механизм нижнего уровня для рисования: массивы вершин (`Vertex arrays`). Фактически, массивы вершин используются внутри всех других классов SFML. Они позволяют более гибко определять 2D-объекты, содержащие столько треугольников, сколько вам нужно. Они даже позволяют рисовать точки или линии.

## Что такое вершина и почему они всегда находятся в массивах?

**Вершина** -- это наименьший графический объект, которым вы можете управлять. Короче говоря, это графическая точка: естественно, у нее есть 2D-позиция (x, y), но также цвет и пара координат текстуры. Позже мы рассмотрим роли этих атрибутов.

Сами по себе вершины (множество вершин) мало что делают. Они всегда группируются в примитивы: **точки** (1 вершина), **линии** (2 вершины),  **треугольники** (3 вершины) или **квадраты** (4 вершины). Затем вы можете объединить несколько примитивов вместе, чтобы создать окончательную геометрию объекта.

Теперь вы понимаете, почему мы всегда говорим о массивах вершин, а не только о вершинах.

## Простой массив вершин

Давайте теперь посмотрим на [sf::Vertex (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Vertex.php) класс. Это просто контейнер, который содержит три публичных атрибута и никаких функций, кроме его конструкторов. Эти конструкторы позволяют создавать вершины из набора параметров, которые вам важны - вам не всегда нужно окрашивать или текстурировать вашу сущность.

```
// объявление новой вершины
sf::Vertex vertex;

// установка позиции вершины
vertex.position = sf::Vector2f(10.f, 50.f);

// цвет вершины
vertex.color = sf::Color::Red;

// координаты текстуры
vertex.texCoords = sf::Vector2f(100.f, 100.f);
```

... или, используя правильный конструктор:

```
sf::Vertex vertex(sf::Vector2f(10.f, 50.f), sf::Color::Red, sf::Vector2f(100.f, 100.f));
```

Теперь давайте определим примитив. Помните, примитив состоит из нескольких вершин, поэтому нам нужен массив вершин (`vertex array`). SFML обеспечивает простую оболочку для этого: [sf::VertexArray (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1VertexArray.php). Он обеспечивает семантику массива (аналогично std::vector), а также сохраняет тип примитива, который определяют его вершины.


```
// создание массива из трёх вершин, которые определяют (опитсывают) треугольный примитив
sf::VertexArray triangle(sf::Triangles, 3);

// установка позиций для точек (вершин) треугльника
triangle[0].position = sf::Vector2f(10.f, 10.f);
triangle[1].position = sf::Vector2f(100.f, 10.f);
triangle[2].position = sf::Vector2f(100.f, 100.f);

// установка цвета для точек (вершин) треугльника
triangle[0].color = sf::Color::Red;
triangle[1].color = sf::Color::Blue;
triangle[2].color = sf::Color::Green;

// здесь нет координат текстуры, мы увидим это позже
```

Ваш треугольник готов, и теперь вы можете его нарисовать. Рисование массива вершин может быть выполнено аналогично рисованию любого другого объекта SFML, используя функцию `draw`:
```
window.draw(triangle);
```

<p align="center">
  <img src="https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-triangle.png">
</p>

Вы можете видеть, что цвет вершин интерполируется для заполнения примитива. Это хороший способ создания градиентов.

Обратите внимание, что вам не нужно использовать [sf::VertexArray (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1VertexArray.php) класс. Он просто определен для удобства, это не что иное, как `std::vector<sf::Vertex>` вместе с `sf::PrimitiveType`. Если вам нужна большая гибкость или статический массив, вы можете использовать собственное хранилище. Затем вы должны использовать перегрузку draw функции, которая принимает указатель на вершины, количество вершин и тип примитива.


```
std::vector<sf::Vertex> vertices;
vertices.push_back(sf::Vertex(...));
...

window.draw(&vertices[0], vertices.size(), sf::Triangles);
```

```
sf::Vertex vertices[2] =
{
    sf::Vertex(...),
    sf::Vertex(...)
};

window.draw(vertices, 2, sf::Lines);
```

... некоторая часть не переведена ...
