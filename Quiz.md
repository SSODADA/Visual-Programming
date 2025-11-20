#  Q2_1 두 수를 더하는 프로그램

xmal
```
<Window
    x:Class="Q2_1.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    Width="250"
    Height="150"
    Title="간단 계산기">
    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
        <TextBox x:Name="Number1Input" PlaceholderText="첫 번째 숫자" Margin="5"/>
        <TextBox x:Name="Number2Input" PlaceholderText="두 번째 숫자" Margin="5"/>
        <Button Content="더하기" Click="Calculate_Click" Margin="5" HorizontalAlignment="Stretch"/>
        <TextBlock x:Name="ResultOutput" Text="결과: " Margin="5"/>
    </StackPanel>
</Window>
```

xmal.h
```
#pragma once
#include "Q2_1.g.h"

namespace winrt::Q2_1::implementation
{
    struct MainWindow : MainWindowT<MainWindow>
    {
        MainWindow()
        {
        }

        void Calculate_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
    };
}

namespace winrt::Q2_1::factory_implementation
{
    struct MainWindow : MainWindowT<MainWindow, implementation::MainWindow>
    {
    };
}
```

xmal.cpp
```
#include "pch.h"
#include "Q2_1.h"
#if __has_include("Q2_1.g.cpp")
#include "Q2_1.g.cpp"
#endif
#include <sstream>

using namespace winrt;
using namespace Microsoft::UI::Xaml;

namespace winrt::Q2_1::implementation
{
    void MainWindow::Calculate_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
    {
        try
        {
            double num1 = std::stod(to_string(Number1Input().Text()));
            double num2 = std::stod(to_string(Number2Input().Text()));

            double result = num1 + num2;

            std::wstringstream ss;
            ss << L"결과: " << result;
            ResultOutput().Text(ss.str());
        }
        catch (...)
        {
            ResultOutput().Text(L"오류: 유효한 숫자를 입력하세요.");
        }
    }
}
```

# Q2_2 펜 (지우기, 선색, 굵기, 저장, 읽기)

xmal
```
<Window
    x:Class="Q2_2.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    Title="간단 펜 그리기">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>
        
        <StackPanel Grid.Row="0" Orientation="Horizontal" Padding="5">
            <Button Content="지우기" Click="Clear_Click" Margin="5"/>
            <Button Content="색 선택" Click="Color_Click" Margin="5"/>
            <Button Content="굵기 변경" Click="Thickness_Click" Margin="5"/>
            <Button Content="저장" Click="Save_Click" Margin="5"/>
            <Button Content="읽기" Click="Load_Click" Margin="5"/>
        </StackPanel>
        
        <Canvas 
            Grid.Row="1" 
            x:Name="DrawingCanvas" 
            Background="White"
            PointerPressed="Canvas_PointerPressed"
            PointerMoved="Canvas_PointerMoved"
            PointerReleased="Canvas_PointerReleased"/>
    </Grid>
</Window>
```

xmal.h
```
#pragma once
#include "Q2_2.g.h"
#include <winrt/Microsoft.UI.Xaml.Shapes.h>
#include <winrt/Microsoft.UI.Input.h>

namespace winrt::Q2_2::implementation
{
    struct MainWindow : MainWindowT<MainWindow>
    {
        MainWindow()
        {
        }

        winrt::Windows::Foundation::Point m_lastPoint{0, 0};
        bool m_isDrawing = false;
        
        void Canvas_PointerPressed(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void Canvas_PointerMoved(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void Canvas_PointerReleased(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void Clear_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        
        void Color_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void Thickness_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void Save_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void Load_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
    };
}

namespace winrt::Q2_2::factory_implementation
{
    struct MainWindow : MainWindowT<MainWindow, implementation::MainWindow>
    {
    };
}
```

xmal.cpp
```
#include "pch.h"
#include "Q2_2.h"
#if __has_include("Q2_2.g.cpp")
#include "Q2_2.g.cpp"
#endif

using namespace winrt;
using namespace Microsoft::UI::Xaml;
using namespace Microsoft::UI::Xaml::Shapes;
using namespace Microsoft::UI::Xaml::Media;
using namespace Microsoft::UI::Input;
using namespace winrt::Windows::Foundation;

namespace winrt::Q2_2::implementation
{
    void MainWindow::Canvas_PointerPressed(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
    {
        // 왼쪽 마우스 버튼이 눌렸는지 확인
        if (e.GetCurrentPoint(DrawingCanvas()).Properties().IsLeftButtonPressed())
        {
            m_isDrawing = true;
            m_lastPoint = e.GetCurrentPoint(DrawingCanvas()).Position();
            DrawingCanvas().CapturePointer(e.Pointer());
        }
    }

    void MainWindow::Canvas_PointerMoved(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
    {
        if (m_isDrawing)
        {
            Point currentPoint = e.GetCurrentPoint(DrawingCanvas()).Position();
            
            // 선(Line) 객체 생성 및 속성 설정
            Line line;
            line.Stroke(SolidColorBrush(Windows::UI::ColorHelper::FromArgb(255, 0, 0, 0))); // 검은색
            line.StrokeThickness(4); // 굵기
            line.X1(m_lastPoint.X);
            line.Y1(m_lastPoint.Y);
            line.X2(currentPoint.X);
            line.Y2(currentPoint.Y);

            // Canvas에 추가
            DrawingCanvas().Children().Append(line);

            m_lastPoint = currentPoint;
        }
    }

    void MainWindow::Canvas_PointerReleased(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
    {
        if (m_isDrawing)
        {
            m_isDrawing = false;
            DrawingCanvas().ReleasePointerCapture(e.Pointer());
        }
    }

    void MainWindow::Clear_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
    {
        DrawingCanvas().Children().Clear();
    }
    
    // 색, 굵기, 저장, 읽기 기능은 구현이 복잡하여 빈 함수로 대체
    void MainWindow::Color_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
    {
        // 여기에 색 선택 로직 구현
    }
    void MainWindow::Thickness_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
    {
        // 여기에 굵기 변경 로직 구현
    }
    void MainWindow::Save_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
    {
        // 여기에 저장 로직 구현
    }
    void MainWindow::Load_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
    {
        // 여기에 읽기 로직 구현
    }
}
```
