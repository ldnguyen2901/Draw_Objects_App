<h1>Draw Objects App</h1>

### Mục lục
[1. Đề bài](#1-đề-bài)

[2. Sơ đồ lớp](#2-sơ-đồ-lớp)

[3. Thiết kế lớp](#3-thiết-kế-lớp)
- [3.1 Lớp điểm (class Point)](#31-lớp-điểm-class-point)
- [3.2 Lớp đối tượng (class _Shape)](#32-lớp-đổi-tượng-class-_shape)
    - [3.2.1 Xây dựng lớp](#321-xây-dựng-lớp)
    - [3.2.2 Thuật toán tìm phần giao giữa hai đối tượng hình bất kỳ](#322-thuật-toán-tìm-phần-giao-giữa-hai-đối-tượng-hình-bất-kỳ)
- [3.3 Lớp hình Ellipse (class _Ellipse)](#33-lớp-hình-ellipse-class-_ellipse)
    - [3.3.1 Xây dựng lớp](#331-xây-dựng-lớp)
    - [3.3.2 Thuật toán kiểm tra vị trí tương đối giữa một điểm và hình Ellipse](#332-thuật-toán-kiểm-tra-vị-trí-tương-đối-giữa-một-điểm-và-hình-ellipse)

=======================================================
<a name='1Debai'></a>  
# 1. **Đề bài**
Cho các loại đối tượng: hình tròn, hình ellipse, đa giác, hình chữ nhật, hình vuông, hình tam giác. Viết chương trình ứng dụng cho phép tạo (hoặc nhập) hai hình thuộc một trong các hình kể trên. In thông báo cho biết hai hình có giao nhau không, nếu có tô màu phần giao và tô đậm đường biên của phần giao. Người sử dụng có thể bấm các phím mũi tên để di chuyển một trong hai hình, phím +, −, để phóng to thu nhỏ một trong hai hình. Có thể lập trình trong môi trường Window. Sử dụng đa hình để giải quyết vấn đề trên.

<a name='2Sodolop'></a>  

# 2. **Sơ đồ lớp**
Dựa vào yêu cầu bài tập lớn, ta xây dựng được các lớp có sơ đồ như sau:

![](Pic/Sodolop.png)

<a name='3Thietkelop'></a>  
# 3. Thiết kế lớp

<a name='31lopdiem'></a> 
## 3.1 Lớp điểm (class Point)

```c++
class Point {
  double x, y;
  public:
    // constructor and destructor
    Point(double xx = 0, double yy = 0): x(xx), y(yy) {}
    ~Point() {}
  // setter
  void set(double xx, double yy) {
    x = xx;
    y = yy;
  }
  // getter
  double getX() const {
    return x;
  }
  double getY() const {
    return y;
  }
  // operation
  void move(double dx, double dy) {
    x += dx;
    y += dy;
  }
  void scale(Point center, double tile);
  void rotate(Point T, double rad);
  // paint
  void mark(HDC hdc, int size = 5, COLORREF c = RGB(0, 0, 0));
  // other method
  double distance(Point a) const;
  Point setVector(Point b) const;
};

```

Trong đó

```C++
void move(double dx, double dy);

void scale(Point center, double tile);

void rotate(Point T, double rad);
```

là các phương thức thực hiện các thao tác tịnh tiến, vị tự, phép quay tâm _T_ (đơn vị độ) của một điểm.

```c++
void mark(HDC hdc, int size = 5, COLORREF c = RGB(0, 0, 0));
```

là phương thức thực hiện đánh dấu (+) một điểm trên cửa sổ ứng dụng.

```c++
double distance(Point a) const;
```

là phương thức thực hiện việc tính khoảng cách giữa hai điểm. Khoảng cách giữa hai điểm $A(x_{1} ,y_{1})$ và $B(x_{2},y_{2})$ được tính thông qua công thức:

$$d = \sqrt{(x_{2}-x_{1})^2 + (y_{2}-y_{1})^2}$$

```c++
Point setVector(Point b) const;
```

là phương thức thực hiện lấy vector giữa hai điểm. Vector $\vec{AB}$ được xác định bằng

$$\vec{AB} = (x_{B}-x_{A};y_{B}-y_{A})$$

<a name ="32lopdoituong"></a>
## 3.2 Lớp đổi tượng (class _Shape)

<a name ="321xaydunglop"></a>
### 3.2.1 Xây dựng lớp

Ta xây dựng một lớp cơ sở trừu tượng (class \_Shape)

```c++
class _Shape {
  public: virtual void move(double dx, double dy) = NULL;
  virtual void scale(double s) = NULL;
  virtual void rotate(double rad) = NULL;
  virtual bool isPointInside(Point p) const = NULL;
  virtual bool isPointInLine(Point p) const = NULL;
  virtual bool intersection(_Shape * aS, HDC hdc, COLORREF c = RGB(156, 200, 30)) const = NULL;
  virtual void draw(HDC hdc, COLORREF c = RGB(0, 100, 100)) const = NULL;
};
```

Trong đó

```C++
virtual bool intersection(_Shape* aS, HDC hdc, COLORREF c = RGB(156, 200, 30)) const = NULL;
```

là phương thức ảo thuần tuý thực hiện việc kiểm tra hai đối tượng hình học đang xét có giao nhau hay không. Với phương thức này, ta có thuật toán tổng quát tìm phần giao cho hai đối tượng hình học bất kỳ.

<a name ="322thuattoantimphangiaogiua2doituonghinhbatky"></a>
### 3.2.2 Thuật toán tìm phần giao giữa hai đối tượng hình bất kỳ

Để tìm phần giao giữa hai đối tượng bất kỳ, ta sẽ xét các điểm trên hình chữ nhật nhỏ nhất chứa một trong hai đối tượng. Nếu điểm đang xét đều nằm trong cả hai đối tượng thì ta sẽ thay đổi màu của điểm đó bằng hàm $SetPixel()$. Tương tự cho phần biên của phần giao, nếu điểm đang xét thuộc cả ba trường hợp sau thì ta sẽ vẽ một hình tròn bán kính 0.5 tại điểm đó:

- Nếu điểm nằm trên biên của đối tượng 1 và nằm bên trong đối tượng 2,
- Nếu điểm nằm bên trong đối tượng 1 và nằm trên biên của đối tượng 2,
- Nếu điểm nằm trên biên của cả hai dối tượng.

Sau khi xét hết hình chữ nhật ta sẽ có phần giao cần tìm với phần viền được tô đậm.

<a name ="33lophinhellipse"></a>
## 3.3 Lớp hình Ellipse (class _Ellipse)

<a name ="331xaydunglop"></a>
## 3.3.1 Xây dựng lớp

```C++
class _Ellipse: public _Shape {
  protected: Point T;
  double xR,
  yR;
  public:
    // constructor and destructor
    _Ellipse(double xT = 700, double yT = 300, double a = 200, double b = 100): 
    T(xT, yT),xR(a),yR(b) {};
  ~_Ellipse() {}
  // setter
  void set(double xT, double yT) {
    T = Point(xT, yT);
    xR = 0;
    yR = 0;
  }
  // operation
  void move(double dx, double dy) override {
    T.move(dx, dy);
  }
  void scale(double s) override {
    xR *= s;
    yR *= s;
  }
  void rotate(double rad) override {
    if (rad == 90 || rad == -90) {
      double temp = xR;
      xR = yR;
      yR = temp;
    }
  }
  void draw(HDC hdc, COLORREF c = RGB(0, 100, 100)) const override {
    Ellipse(hdc, T.getX() - xR, T.getY() - yR, T.getX() + xR, T.getY() + yR);
  }
  // check
  bool isPointInside(Point p) const override;
  bool isPointInLine(Point p) const override;
  // intersection
  bool intersection(_Shape * aS, HDC hdc, COLORREF c = RGB(156, 200, 30))
  const override;
};
```
Trong đó
 - class <span style="color:rgb(96, 158, 187)">_Ellipse</span> là lớp con được kế thừa từ lớp cha class <span style="color:rgb(96, 158, 187)">_Shape</span>.

 ```C++
void move(double dx, double dy) override;
 ``` 
là phương thức tịnh tiến hình Ellipse bằng cách tịnh tiến tâm T của hình.
 ```C++
void scale(double s) override;
```
là phương thức phóng to (thu nhỏ) hình Ellipse bằng cách nhân độ dài hai bán trục
với tỷ lệ s.
```C++
• void rotate(double rad) override;
```
là phương thức thực hiện quay hình Ellipse với góc quay có đơn vị độ. Ở phương
thức này, ta chỉ xét phép quay với góc $90^\circ$ vì ta sử dụng hàm vẽ Ellipse mặc định
của môi trường Windows nên chỉ có thể vẽ các hình Ellipse có hai bán trục song
song với hai trục toạ độ.
```C++
• bool isPointInside(Point p) const override;
```
là phương thức kiểm tra một điểm nằm bên trong hình Ellipse (không tính phần
biên).
```C++
• bool isPointInLine(Point p) const override;
```
là phương thức kiểm tra một điểm nằm trên đường biên của hình Ellipse.
```C++
bool intersection(_Shape* aS, HDC hdc, COLORREF c = RGB(156, 200, 30)) const override;
```
là phương thức kiểm tra hình Ellipse và một đối tượng hình học giao nhau.

<a name="332thuattoankiemtravitrituongdoigiuamotdiemvahinhellipse"></a>
### 3.3.2 Thuật toán kiểm tra vị trí tương đối giữa một điểm và hình Ellipse
Để kiểm tra một điểm $P(x_{P}, y_{P})$ có nằm trong hình Ellipse đang xét hay không, ta sử
dụng phương trình chính tắc của hình Ellipse
$$(\frac{x - x_{0}}{a})^2+(\frac{y - y_{0}}{b})^2=1 {(1)}$$
