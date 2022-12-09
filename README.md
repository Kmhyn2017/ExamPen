# ExamPenAndReadWrite
---
## Xaml

#### 윈도우 제목   
```
<TextBlock
                Margin="10,0,0,0"
                VerticalAlignment="Center"
                Text="Andong Nation University"
                TextWrapping="NoWrap"/>
```
윈도우 제목을 만들위해서 다음과 같은 텍스트블록에 텍스트 작성합니다.   
---
#### 메뉴 추가
```
<MenuBar>
                    <MenuBarItem Title="File">
                        <MenuFlyoutItem Text="New File" Click="MenuFlyoutItem_Click" />
                        <MenuFlyoutItem Text="Save" Click="MenuFlyoutItem_Click_1" />
                        <MenuFlyoutItem Text="Load" Click="MenuFlyoutItem_Click_2" />
                        <MenuFlyoutItem Text="Exit" Click="MenuFlyoutItem_Click_3" />
                    </MenuBarItem>
                </MenuBar>
```
메뉴 바에 서브 버튼을 추가하여 첫 버튼 새로운 파일 생성, 두번째 버튼 저장,   
세번째 버튼 Load, 네번쨰 버튼 Exit   

슬라이더를 추가하고   

#### 옆 버튼에 읽기, 저장, 지우기, 감추기, 보이기 추가한다.
```
<Slider AutomationProperties.Name="simple slider" Width="200" Grid.Column="0" Grid.Row="0"
            ValueChanged="Slider_ValueChanged"/>
            <Button x:Name="myWrite" Click="myWrite_Click">Write</Button>
            <Button x:Name="myRead" Click="myRead_Click">Read</Button>
            <Button x:Name="myClear" Click="myClear_Click">Clear</Button>
            <Button x:Name="CloseColor" Click="CloseColor_Click">감추기</Button>
            <Button x:Name="OpenColor" Click="OpenColor_Click">보이기</Button>
```
#### 펜을 그리기 위한 캔버스와 컬러피커도 추가한다,
```
<canvas:CanvasControl Grid.Column="0" Grid.Row="1"
        PointerEntered="CanvasControl_PointerEntered"
        PointerPressed="CanvasControl_PointerPressed"
        PointerReleased="CanvasControl_PointerReleased"
        PointerMoved="CanvasControl_PointerMoved"
        Draw="CanvasControl_Draw" ClearColor="CornflowerBlue"/>
        <ColorPicker Grid.Column="1" Grid.Row="1" 
        x:Name="ColorPicker"
        ColorChanged="ColorPicker_ColorChanged"
        ColorSpectrumShape="Ring"
        IsMoreButtonVisible="False"
        IsColorSliderVisible="True"
        IsColorChannelTextInputVisible="True"
        IsHexInputVisible="True"
        IsAlphaEnabled="False"
        IsAlphaSliderVisible="True"
        IsAlphaTextInputVisible="True" />
```
## Xaml.h
---
컬러피커를 구현하기 위해 변수 선언과 백터 선언을 해준다.   
```
        bool flag;
        float px, py;
        float mySize;
        std::vector<float> vx;
        std::vector<float> vy;
        std::vector<struct winrt::Windows::UI::Color> col;
        std::vector<float> size;
```
## Xaml.cpp
---
캔버스를 그리기 위해 다음과 같은 기능을 추가한다.
```
struct winrt::Windows::UI::Color myCol = winrt::Microsoft::UI::Colors::Green();
void winrt::ExamPen::implementation::MainWindow::CanvasControl_Draw(winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasControl const& sender, winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasDrawEventArgs const& args)
{
    CanvasControl canvas = sender.as<CanvasControl>();
    int n = vx.size();
    if (n <= 2) return;
    for (int i = 1; i < n; i++) {
        if (vx[i] == 0.0 && vy[i] == 0.0) {
            i++; continue;
        }
        args.DrawingSession().DrawLine(vx[i - 1], vy[i - 1], vx[i], vy[i], col[i], size[i]);
        args.DrawingSession().FillCircle(vx[i - 1], vy[i - 1], size[i] / 2, col[i]);
        args.DrawingSession().FillCircle(vx[i], vy[i], size[i] / 2, col[i]);
    }
}
```
캔버스에 펜을 그릴때 x, y에 따라서 그림을 그리기 위해 기능을 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::CanvasControl_PointerMoved(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    CanvasControl canvas = sender.as<CanvasControl>();
    px = e.GetCurrentPoint(canvas).Position().X;
    py = e.GetCurrentPoint(canvas).Position().Y;
    if (flag) {
        vx.push_back(px);
        vy.push_back(py);
        col.push_back(myCol);
        size.push_back(mySize);
        canvas.Invalidate();
    }
}
```
색상이 변경됐을 떄 색을 바꿀 수 있도록 기능을 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::ColorPicker_ColorChanged(winrt::Microsoft::UI::Xaml::Controls::ColorPicker const& sender, winrt::Microsoft::UI::Xaml::Controls::ColorChangedEventArgs const& args)
{
    myCol = args.NewColor();
}
```
펜이 캔버스에서 떨어졌을때 위치를 갖는 기능을 추가합니다.
```
void winrt::ExamPen::implementation::MainWindow::CanvasControl_PointerReleased(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    flag = false;
    px = py = 0.0;
    vx.push_back(px);
    vy.push_back(py);
    col.push_back(myCol);
    size.push_back(mySize);
}
```
슬라이더 값에 따라서 펜 굵기를 변경하기 위해 기능을 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::Slider_ValueChanged(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Controls::Primitives::RangeBaseValueChangedEventArgs const& e)
{
    mySize = e.NewValue();
}
```
캔버스 저장 기능을 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::myWrite_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    FILE* fw;
    errno_t err;
    err = fopen_s(&fw, "C:/Users/jcshim/my.txt", "w");// +, ccs = UTF - 8");
    if (err == 0)
    {
        MessageBox(NULL, L"The file was opened\n", L"hi1", NULL);
        int num = vx.size();
        fprintf_s(fw, "%d\n", num);
        for (int i = 0; i < num; i++) {
            fprintf_s(fw, "%f %f %d %d %d %d %f\n", vx[i], vy[i],
                col[i].A, col[i].B, col[i].G, col[i].R, size[i]);
        }
        fclose(fw);
    }
    else  MessageBox(NULL, L"The file was not opened\n", L"hi2", NULL);
}
```
캔버스 그림을 지우는 기능을 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::myClear_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    vx.clear();
    vy.clear();
    col.clear();
    size.clear();
    flag = false;
    px = 100;
    py = 100;
    mySize = 16;
}
```
캔버스 읽기 기능을 저장한다.
```
void winrt::ExamPen::implementation::MainWindow::myRead_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    FILE* fr;
    errno_t err;
    err = fopen_s(&fr, "C:/Users/jcshim/my.txt", "r");// +, ccs = UTF - 8");
    if (err == 0)
    {
        //  MessageBox(NULL, L"The file was opened\n", L"hi1", NULL);
        vx.clear();
        vy.clear();
        col.clear();
        size.clear();
        int num;
        float vx1, vy1, size1;
        int colA, colB, colG, colR;

        fscanf_s(fr, "%d", &num);
        for (int i = 0; i < num; i++) {
            fscanf_s(fr, "%f %f %hhu %hhu %hhu %hhu %f ", &vx1, &vy1, &colA, &colB, &colG, &colR, &size1);
            vx.push_back(vx1);
            vy.push_back(vy1);
            myCol.A = colA;
            myCol.B = colB;
            myCol.G = colG;
            myCol.R = colR;
            col.push_back(myCol);
            size.push_back(size1);
        }
        fclose(fr);
        //MessageBox(NULL, L"The file closed\\n", L"hi1", NULL);
        CanvasControl_PointerReleased(NULL, NULL);
    }
    else  MessageBox(NULL, L"The file was not opened\\n", L"hi2", NULL);
}
```
메뉴바 기능 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::MenuFlyoutItem_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    vx.clear();
    vy.clear();
    size.clear();
    col.clear();
}
```
서브 버튼 새로운 버튼, 저장, Load, Exit기능을 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::MenuFlyoutItem_Click_1(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    FILE* fp;
    fopen_s(&fp, "C:/Users/jcshim/Desktop/1.txt", "w");
    if (fp)
    {
        int n = vx.size();
        for (int i = 0; i < n; i++)
        {
            fprintf(fp, "%f %f %d %d %d %d %f ", vx[i], vy[i], col[i].R, col[i].G, col[i].B, col[i].A, size[i]);
        }
        fclose(fp);
    }
}


void winrt::ExamPen::implementation::MainWindow::MenuFlyoutItem_Click_2(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    FILE* fp;
    fopen_s(&fp, "C:/Users/jcshim/Desktop/1.txt", "r");
    if (fp)
    {
        vx.clear(); vy.clear(); size.clear(); col.clear();

        float x, y, s;
        int r, g, b, a;
        while (EOF != fscanf_s(fp, "%f %f %d %d %d %d %f", &x, &y, &r, &g, &b, &a, &s))
        {
            vx.push_back(x);
            vy.push_back(y);
            Windows::UI::Color tCol;
            tCol.R = r; tCol.G = g; tCol.B = b; tCol.A = a;
            col.push_back(tCol);
            size.push_back(s);
        }
        fclose(fp);
    }
}


void winrt::ExamPen::implementation::MainWindow::MenuFlyoutItem_Click_3(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    this->Close();
}
```
컬러피커 보이기위해 기능을 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::OpenColor_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    ColorPicker().IsColorSpectrumVisible(true);
}
```
컬러피커 감추기위해 기능을 추가한다.
```
void winrt::ExamPen::implementation::MainWindow::CloseColor_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    ColorPicker().IsColorSpectrumVisible(false);
}
```

## 구현
---
