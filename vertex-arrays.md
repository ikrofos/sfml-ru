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
```
#include <SFML/Graphics.hpp>
#include <SFML/Graphics/RenderWindow.hpp>

int main()
{	

	sf::RenderWindow window(sf::VideoMode(600, 600), "Triangle");

	sf::VertexArray triangle(sf::Triangles, 3);

	triangle[0].position = sf::Vector2f(10.f, 10.f);
	triangle[1].position = sf::Vector2f(400.f, 10.f);
	triangle[2].position = sf::Vector2f(400.f, 400.f);

	triangle[0].color = sf::Color::Red;
	triangle[1].color = sf::Color::Blue;
	triangle[2].color = sf::Color::Green;

	while (window.isOpen())
	{
		sf::Event event;
		while (window.pollEvent(event))
		{
			if (event.type == sf::Event::Closed)
				window.close();
		}
		window.clear(sf::Color(250, 220, 100, 0));

		window.draw(triangle);

		window.display();
	}
}
```
<p align="center">
  <img src="https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-triangle.png">
</p>

Вы можете видеть, что цвет вершин интерполируется для заполнения примитива. Это хороший способ создания градиентов.

Обратите внимание, что вам не нужно использовать [sf::VertexArray (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1VertexArray.php) класс. Он просто определен для удобства, это не что иное, как `std::vector<sf::Vertex>` вместе с `sf::PrimitiveType`. 
> Для добавления нового элемента в конец вектора используется метод `push_back()` класса `std::vector`.
```
std::vector<sf::Vertex> vertices;
vertices.push_back(sf::Vertex(...));
...

window.draw(&vertices[0], vertices.size(), sf::Triangles);
```
Если вам нужна большая гибкость или статический массив, вы можете использовать собственное хранилище. Затем вы должны использовать перегрузку draw функции, которая принимает указатель на вершины (`vertices`), количество вершин (`2`) и тип примитива (`sf::Lines`).

```
sf::Vertex vertices[2] =
{
    sf::Vertex(...),
    sf::Vertex(...)
};

window.draw(vertices, 2, sf::Lines);
```
## Примитивные типы

Давайте ненадолго остановимся и посмотрим, какие примитивы вы можете создать. Как объяснялось выше, вы можете определить самые основные 2D-примитивы: точка, линия, треугольник и четырехугольник (четырехугольник существует просто для удобства, внутри видеокарта разбивает его на два треугольника). Существуют также «связанные» варианты этих примитивных типов, которые позволяют разделять вершины между двумя последовательными примитивами. Это может быть полезно, потому что последовательные примитивы часто каким-то образом связаны.

Давайте посмотрим на полный список:

| Примитив      | Описание      | Пример |
| -             | -             |-       |
| sf::Points  | Набор несвязанных точек. Эти точки не имеют толщины: они всегда будут занимать один пиксель, независимо от текущего преобразования и вида.  | ![](https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-points.png) |
| sf::Lines  | 	Набор несвязанных линий. Эти линии не имеют толщины: они всегда будут иметь ширину в один пиксель, независимо от текущего преобразования и вида.  | ![](https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-lines.png) |
| sf::LineStrip  | 	Набор связанных линий. Конечная вершина одной линии используется как начальная вершина следующей.  | ![](https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-line-strip.png) |
| sf::Triangles  | 	Набор несвязанных треугольников.  | ![](https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-triangles.png) |
| sf::TriangleStrip  | 	Набор связанных треугольников. У каждого треугольника две последние вершины совпадают со следующим.  | ![](https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-triangle-strip.png) |
| sf::TriangleFan  | 	Набор треугольников, соединенных с центральной точкой. Первая вершина - это центр, затем каждая новая вершина определяет новый треугольник, используя центр и предыдущую вершину.  | ![](https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-triangle-fan.png) |
| sf::Quads  | 	Набор несвязанных четырехугольников. 4 точки каждой четверки должны быть определены последовательно, либо по часовой стрелке, либо против часовой стрелки.  | ![](https://www.sfml-dev.org/tutorials/2.5/images/graphics-vertex-array-quads.png) |

## Текстурирование

Как и другие объекты SFML, массивы вершин также могут быть текстурированы. Для этого вам нужно будет изменить атрибут texCoords класса [sf::Vertex (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Vertex.php). Этот атрибут определяет, какой пиксель текстуры сопоставлен с вершиной.

```
// создание четырёхугольника
sf::VertexArray quad(sf::Quads, 4);

// определим его как квадрат расположенный по координатам (10, 10) и размером 100x100
quad[0].position = sf::Vector2f(10.f, 10.f);
quad[1].position = sf::Vector2f(110.f, 10.f);
quad[2].position = sf::Vector2f(110.f, 110.f);
quad[3].position = sf::Vector2f(10.f, 110.f);

// define its texture area to be a 25x50 rectangle starting at (0, 0)
quad[0].texCoords = sf::Vector2f(0.f, 0.f);
quad[1].texCoords = sf::Vector2f(25.f, 0.f);
quad[2].texCoords = sf::Vector2f(25.f, 50.f);
quad[3].texCoords = sf::Vector2f(0.f, 50.f);
```

```
Координаты текстуры определяются в пикселях (как и textureRect спрайтов и фигур). Они не нормализованы (от 0 до 1), как могли ожидать люди, привыкшие к программированию OpenGL.
```

Массивы вершин - это низкоуровневые объекты, они имеют дело только с геометрией и не хранят дополнительных атрибутов, таких как текстура. Чтобы нарисовать массив вершин с текстурой, вы должны передать его непосредственно в функцию отрисовки `draw`:

```
sf::VertexArray vertices;
sf::Texture texture;

...
// используется конструктор 
// sf::RenderStates::RenderStates (const Texture * theTexture)	

window.draw(vertices, &texture);
```


Это краткая версия, если вам нужно передать другие состояния рендеринга (например, режим наложения или преобразование), вы можете использовать явную версию, которая принимает объект [ sf::RenderStates (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1RenderStates.php):

```
sf::VertexArray vertices;
sf::Texture texture;

...

sf::RenderStates states;
states.texture = &texture;

window.draw(vertices, states);
```

## Трансформация массива вершин

Трансформация массива вершин похожа на текстурирование. Трансформация не сохраняется в массиве вершин, вы должны передать его функции отрисовки `draw()`.

```
sf::VertexArray vertices;
sf::Transform transform;

...

window.draw(vertices, transform);
```

Или, если вам нужно передать другие состояния рендеринга:

```
sf::VertexArray vertices;
sf::Transform transform;

...

sf::RenderStates states;
states.transform = transform;

window.draw(vertices, states);
```

Чтобы узнать больше о преобразованиях и классе [sf::Transform (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Transform.php), вы можете прочитать [руководство по преобразованию сущностей](https://www.sfml-dev.org/tutorials/2.5/graphics-transform.php).

## Создание объекта, подобного SFML

Теперь, когда вы знаете, как определить свою собственную текстурированную / окрашенную / трансформированную сущность, было бы неплохо обернуть ее в класс в стиле SFML? К счастью, SFML делает это легко для вас, предоставляя базовые классы [sf::Drawable (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Drawable.php) и [sf::Transformable](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Transformable.php). Эти два класса являются базой для встроенных сущностей SFML таких как [sf::Sprite](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Sprite.php), [sf::Text](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Text.php) и [sf::Shape](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Shape.php).

[sf::Drawable (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Drawable.php) является интерфейсом: он объявляет единственную [чистую виртуальную функцию](https://prog-cpp.ru/cpp-virtual/#:~:text=%D0%A7%D0%B8%D1%81%D1%82%D0%B0%D1%8F%20%D0%B2%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F%20%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F%20%E2%80%94%20%D1%8D%D1%82%D0%BE%20%D0%BC%D0%B5%D1%82%D0%BE%D0%B4%20%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D0%B0%2C%20%D1%82%D0%B5%D0%BB%D0%BE%20%D0%BA%D0%BE%D1%82%D0%BE%D1%80%D0%BE%D0%B3%D0%BE%20%D0%BD%D0%B5%20%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%BE.&text=virtual%20void%20func()%20%3D%200,%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D0%B8%20%D0%BD%D0%B0%20%D0%B1%D0%BE%D0%BB%D0%B5%D0%B5%20%D0%BF%D0%BE%D0%B7%D0%B4%D0%BD%D0%B8%D0%B9%20%D1%81%D1%80%D0%BE%D0%BA.) и не имеет ни членов, ни конкретных функций. Наследование от [sf::Drawable (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Drawable.php) позволяет рисовать экземпляры вашего класса так же, как классы SFML (используя метод класса void draw(const Drawable& drawable, const RenderStates& states = RenderStates::Default);):

_sf::Drawable заставляет инициализовать (virtual void draw(RenderTarget& target, RenderStates states) const = 0;)_ чтобы можно было вызывать entity.draw(window).
В sf::RenderTarget определена функция void draw(const Drawable& drawable, const RenderStates& states = RenderStates::Default); работающая с объектом sf::Drawable__
```
сlass MyEntity : public sf::Drawable
{
private:

    virtual void draw(sf::RenderTarget& target, sf::RenderStates states) const;
};

MyEntity entity;
window.draw(entity); // internally calls entity.draw
```

Обратите внимание, что это не обязательно, вы также можете иметь аналогичную функцию отрисовки `draw` в своем классе и просто вызывать ее с помощью entity.draw(window). Но другой способ, с базовым классом [sf::Drawable (en)](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Drawable.php), более приятный и последовательный. Это также означает, что если вы планируете хранить массив доступных для рисования объектов, вы можете сделать это без каких-либо дополнительных усилий, поскольку все доступные для рисования объекты (SFML и ваши) являются производными от одного и того же класса.

Другой базовый класс [sf::Transformable](https://www.sfml-dev.org/documentation/2.5.1/classsf_1_1Transformable.php) не имеет виртуальной функции. Наследование автоматически добавляет ту же функцию преобразования к классу, как и другие классы SFML ( setPosition, setRotation, move, scale, ...). Вы можете узнать больше об этом классе в учебнике по [преобразованию сущностей](https://www.sfml-dev.org/tutorials/2.5/graphics-transform.php).

Используя эти два базовых класса и массив вершин (в этом примере мы также добавим текстуру), вот как будет выглядеть типичный SFML-подобный графический класс:

```
class MyEntity : public sf::Drawable, public sf::Transformable
{
public:

    // add functions to play with the entity's geometry / colors / texturing...

private:

    virtual void draw(sf::RenderTarget& target, sf::RenderStates states) const
    {
        // apply the entity's transform -- combine it with the one that was passed by the caller
        states.transform *= getTransform(); // getTransform() is defined by sf::Transformable

        // применить текстуру
        states.texture = &m_texture;

        // you may also override states.shader or states.blendMode if you want

        // отрисовка массива вершин
        target.draw(m_vertices, states);
    }

    sf::VertexArray m_vertices;
    sf::Texture m_texture;
};
```

Затем вы можете использовать этот класс, как если бы он был встроенным классом SFML:

```
MyEntity entity;

// можем трансформировать объект
entity.setPosition(10.f, 50.f);
entity.setRotation(45.f);

// и отрисовать его
window.draw(entity);
```


... некоторая часть не переведена ...
