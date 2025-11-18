# Visual-Programming

## 1. 라디오 버튼을 활용한 프로필 사진 선택 기능

xaml
```
<?xml version="1.0" encoding="utf-8"?>
<Window
    x:Class="Test.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Test"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:muxc="using:Microsoft.UI.Xaml.Controls"
    mc:Ignorable="d">
    <StackPanel>
    <Grid x:Name="Example1">
        <PersonPicture x:Name="personPicture" Height="300" VerticalAlignment="Top" />
    </Grid>
    <Grid>
        <RadioButtons Header="Options:">
            <RadioButton x:Name="ProfileImageRadio" Content="Profile Image" Checked="ProfileImageRadio_Checked"/>
            <RadioButton x:Name="DisplayNameRadio" Content="Display Name" Checked="DisplayNameRadio_Checked"/>
            <RadioButton x:Name="InitialsRadio" Content="Initials" Checked="InitialsRadio_Checked"/>
        </RadioButtons>
    </Grid>
        <TextBlock x:Name="aaa"></TextBlock>
    </StackPanel>
</Window>
```

xaml.h
```
#pragma once

#include "MainWindow.g.h"

namespace winrt::Test::implementation
{
    struct MainWindow : MainWindowT<MainWindow>
    {
        MainWindow()
        {
            // Xaml objects should not call InitializeComponent during construction.
            // See https://github.com/microsoft/cppwinrt/tree/master/nuget#initializecomponent
        }

        int32_t MyProperty();
        void MyProperty(int32_t value);
        void ProfileImageRadio_Checked(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void DisplayNameRadio_Checked(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void InitialsRadio_Checked(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
    };
}

namespace winrt::Test::factory_implementation
{
    struct MainWindow : MainWindowT<MainWindow, implementation::MainWindow>
    {
    };
}
```

xaml.cpp
```
#include "pch.h"
#include "MainWindow.xaml.h"
#if __has_include("MainWindow.g.cpp")
#include "MainWindow.g.cpp"
#endif
#include "winrt/Microsoft.UI.Xaml.Media.Imaging.h"         //image용
#include "string"                                          //string용

using namespace winrt;
using namespace Microsoft::UI::Xaml;
using namespace Microsoft::UI::Xaml::Controls;            //PersonPicture용
using namespace Microsoft::UI::Xaml::Media::Imaging;      //image용
using namespace Microsoft::UI::Xaml::Media;               //image용
using namespace Windows::Foundation;                      //URI용
using namespace std;                                      //편의용 std

// To learn more about WinUI, the WinUI project structure,
// and more about our project templates, see: http://aka.ms/winui-project-info.

namespace winrt::Test::implementation
{
    int32_t MainWindow::MyProperty()
    {
        throw hresult_not_implemented();
    }

    void MainWindow::MyProperty(int32_t /* value */)
    {
        throw hresult_not_implemented();
    }
}

void winrt::Test::implementation::MainWindow::ProfileImageRadio_Checked(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    string st = "You selected Option 1 ";
    aaa().Text(to_hstring(st));

    Image img;
    BitmapImage bimg;
    Uri uri = Uri(to_hstring("https://docs.microsoft.com/windows/uwp/contacts-and-calendar/images/shoulder-tap-static-payload.png"));
    bimg.UriSource(uri);
    img.Source() = bimg;
    personPicture().ProfilePicture(bimg);
}

void winrt::Test::implementation::MainWindow::DisplayNameRadio_Checked(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    string st = "You selected Option 2 ";    // 텍스트 박스에 넣을 문구
    aaa().Text(to_hstring(st));    //텍스트 박스에 문구 출력

    personPicture().ProfilePicture(NULL);  //프로필에 이미지 삭제
    personPicture().Initials(to_hstring(""));  //프로필에 이니셜 삭제
    personPicture().DisplayName(to_hstring("Jane Doe"));  //프로필에 이름 설정
}

void winrt::Test::implementation::MainWindow::InitialsRadio_Checked(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    string st = "You selected Option 3 ";   // 텍스트 박스에 넣을 문구
    aaa().Text(to_hstring(st));  // 텍스트 박스에 문구 출력

    personPicture().ProfilePicture(NULL);  //프로필 이미지 삭제
    personPicture().Initials(to_hstring("SB"));  // 프로필에 이니셜 설정
}
```

## 2. 이미지 콜라주

xaml
```
<Window
    x:Class="App12.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:App12"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
    mc:Ignorable="d">

    <Grid Padding="5">
        <Grid>
            <Grid.RowDefinitions>
                <RowDefinition Height="60" />
                <RowDefinition Height="590" />
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="600" />
                <ColumnDefinition Width="400" />
            </Grid.ColumnDefinitions>

            <Slider AutomationProperties.Name="simple slider" Width="200" Grid.Column="0" Grid.Row="0"
                    ValueChanged="Slider_ValueChanged"/>

            <AppBarToggleButton x:Name="ButtonOn" Icon="Shuffle" IsChecked="True" Label="Enable" 
                    Grid.Column="1" Grid.Row="0"
                    Click="ButtonOn_Click"/>

            <canvas:CanvasControl Grid.Column="0" Grid.Row="1"
                PointerPressed="CanvasControl_PointerPressed"
                PointerReleased="CanvasControl_PointerReleased"
                PointerMoved="CanvasControl_PointerMoved"
                Draw="CanvasControl_Draw" ClearColor="CornflowerBlue"/>
        </Grid>
        <Border x:Name="colorPanel" Visibility="Visible">
            <Grid>
                <Grid.RowDefinitions>
                    <RowDefinition Height="60" />
                    <RowDefinition Height="590" />
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="600" />
                    <ColumnDefinition Width="400" />
                </Grid.ColumnDefinitions>


                <ColorPicker Grid.Column="1" Grid.Row="1" 
                    ColorChanged="ColorPicker_ColorChanged"
                    ColorSpectrumShape="Ring"
                    IsMoreButtonVisible="False"
                    IsColorSliderVisible="True"
                    IsColorChannelTextInputVisible="True"
                    IsHexInputVisible="True"
                    IsAlphaEnabled="False"
                    IsAlphaSliderVisible="True"
                    IsAlphaTextInputVisible="True" />
            </Grid>
        </Border>
    </Grid>
</Window>
```

xaml.h
```
#pragma once

#include "MainWindow.g.h"
#include <winrt/Microsoft.Graphics.Canvas.UI.Xaml.h> // for canvas
#include <winrt/Microsoft.UI.Xaml.Input.h>
#include <winrt/Microsoft.UI.Input.h>

using namespace winrt::Microsoft::UI;
using namespace winrt::Microsoft::Graphics::Canvas::UI::Xaml;

namespace winrt::App12::implementation
{
    struct MainWindow : MainWindowT<MainWindow>
    {
        MainWindow();

        int32_t MyProperty();
        bool IsChecked = true;
        bool flag = false;
        float px = 100, py = 100;
        float mySize = 16;
        std::vector<float> vx;
        std::vector<float> vy;
        std::vector<struct winrt::Windows::UI::Color> col;
        std::vector<float> size;
        void MyProperty(int32_t value);
        void CanvasControl_Draw(winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasControl const& sender, winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasDrawEventArgs const& args);
        void CanvasControl_PointerPressed(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void CanvasControl_PointerMoved(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void ColorPicker_ColorChanged(winrt::Microsoft::UI::Xaml::Controls::ColorPicker const& sender, winrt::Microsoft::UI::Xaml::Controls::ColorChangedEventArgs const& args);
        void CanvasControl_PointerReleased(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void Slider_ValueChanged(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Controls::Primitives::RangeBaseValueChangedEventArgs const& e);
        void ButtonOn_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
    };
}

namespace winrt::App12::factory_implementation
{
    struct MainWindow : MainWindowT<MainWindow, implementation::MainWindow>
    {
    };
}
```

xaml.cpp
```
#include "pch.h"
#include "MainWindow.xaml.h"
#if __has_include("MainWindow.g.cpp")
#include "MainWindow.g.cpp"
#endif

using namespace winrt;
using namespace Microsoft::UI::Xaml;

// To learn more about WinUI, the WinUI project structure,
// and more about our project templates, see: http://aka.ms/winui-project-info.

namespace winrt::App12::implementation
{
    MainWindow::MainWindow()
    {
        InitializeComponent();
    }

    int32_t MainWindow::MyProperty()
    {
        throw hresult_not_implemented();
    }

    void MainWindow::MyProperty(int32_t /* value */)
    {
        throw hresult_not_implemented();
    }


}
struct winrt::Windows::UI::Color myCol = winrt::Microsoft::UI::Colors::Green();
void winrt::App12::implementation::MainWindow::CanvasControl_Draw(winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasControl const& sender, winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasDrawEventArgs const& args)
{
    CanvasControl canvas = sender.as<CanvasControl>();
    int n = vx.size();
    for (int i = 1; i < n; i++) {
        if (vx[i] == 0 && vy[i] == 0) {
            i++; continue;
        }
        args.DrawingSession().DrawLine(vx[i - 1], vy[i - 1], vx[i], vy[i], col[i], size[i]);
        args.DrawingSession().FillCircle(vx[i - 1], vy[i - 1], size[i] / 2, col[i]);
        args.DrawingSession().FillCircle(vx[i], vy[i], size[i] / 2, col[i]);
    }

    //args.DrawingSession().DrawEllipse(px, py, 80, 30, Colors::Black(), 8);
    //args.DrawingSession().DrawText(L"Hello, world!", px-55, py-15, Colors::Yellow());
}

void winrt::App12::implementation::MainWindow::CanvasControl_PointerPressed(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    flag = true;
}
void winrt::App12::implementation::MainWindow::CanvasControl_PointerMoved(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
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
void winrt::App12::implementation::MainWindow::ColorPicker_ColorChanged(winrt::Microsoft::UI::Xaml::Controls::ColorPicker const& sender, winrt::Microsoft::UI::Xaml::Controls::ColorChangedEventArgs const& args)
{
    myCol = args.NewColor();
}

void winrt::App12::implementation::MainWindow::CanvasControl_PointerReleased(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    flag = false;
    px = py = 0.0;
    vx.push_back(px);
    vy.push_back(py);
    col.push_back(myCol);
    size.push_back(mySize);
}

void winrt::App12::implementation::MainWindow::Slider_ValueChanged(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Controls::Primitives::RangeBaseValueChangedEventArgs const& e)
{
    mySize = e.NewValue();
}
void winrt::App12::implementation::MainWindow::ButtonOn_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    if (IsChecked)
    {
        IsChecked = FALSE;
        ButtonOn().Label(L"Disable");

        colorPanel().Visibility(Visibility::Collapsed);
    }
    else
    {
        IsChecked = TRUE;
        ButtonOn().Label(L"Enable");

        colorPanel().Visibility(Visibility::Visible);
    }
}
```

## 3. 메뉴바

 xaml
 ```
 <Window
    x:Class="App3.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:App3"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="40" />
            <RowDefinition Height="60" />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto"/>
        </Grid.ColumnDefinitions>
        <Grid Grid.Row="0" Background="Gainsboro" CornerRadius="10">
            <MenuBar>
                <MenuBarItem Title="File">
                    <MenuFlyoutItem Text="Save" Click="MenuFlyoutItem_Click" Icon="Save">
                        <MenuFlyoutItem.KeyboardAccelerators>
                            <KeyboardAccelerator Modifiers="Control" Key="S" />
                        </MenuFlyoutItem.KeyboardAccelerators>
                    </MenuFlyoutItem>
                </MenuBarItem>
                <MenuBarItem Title="Edit">
                    <MenuFlyoutItem Text="Cut" Click="MenuFlyoutItem_Click_2"  Icon="Cut">
                        <MenuFlyoutItem.KeyboardAccelerators>
                            <KeyboardAccelerator Modifiers="Control" Key="X"/>
                        </MenuFlyoutItem.KeyboardAccelerators>
                    </MenuFlyoutItem>
                    <MenuFlyoutItem Text="Copy" Click="MenuFlyoutItem_Click_3"  Icon="Copy">
                        <MenuFlyoutItem.KeyboardAccelerators>
                            <KeyboardAccelerator Modifiers="Control" Key="C"/>
                        </MenuFlyoutItem.KeyboardAccelerators>
                    </MenuFlyoutItem>
                    <MenuFlyoutItem Text="Paste" Click="MenuFlyoutItem_Click_4"  Icon="Paste">
                        <MenuFlyoutItem.KeyboardAccelerators>
                            <KeyboardAccelerator Modifiers="Control" Key="V"/>
                        </MenuFlyoutItem.KeyboardAccelerators>
                    </MenuFlyoutItem>
                </MenuBarItem>
                <MenuBarItem Title="Help">
                    <MenuFlyoutItem Text="About" Click="MenuFlyoutItem_Click_5" Icon="Help">
                        <MenuFlyoutItem.KeyboardAccelerators>
                            <KeyboardAccelerator Modifiers="Control" Key="I" />
                        </MenuFlyoutItem.KeyboardAccelerators>
                    </MenuFlyoutItem>
                </MenuBarItem>
                <MenuBarItem Title="Option">
                    <MenuFlyoutSubItem Text="On/Off">
                        <RadioMenuFlyoutItem Text="on" Click="RadioMenuFlyoutItem_Click" IsChecked="True"/>
                        <RadioMenuFlyoutItem Text="off" Click="RadioMenuFlyoutItem_Click_1"/>
                    </MenuFlyoutSubItem>
                </MenuBarItem>
                <MenuBarItem Title="File Option">
                    <MenuFlyoutItem Text="open" Click="MenuFlyoutItem_Click_1">
                        <MenuFlyoutItem.KeyboardAccelerators>
                            <KeyboardAccelerator Modifiers="Control" Key="O" />
                        </MenuFlyoutItem.KeyboardAccelerators>
                    </MenuFlyoutItem>
                    <MenuFlyoutSubItem Text="send to">
                        <MenuFlyoutItem Text="Bluetooth" Click="MenuFlyoutItem_Click_6"/>
                        <MenuFlyoutItem Text="Email" Click="MenuFlyoutItem_Click_7"/>
                    </MenuFlyoutSubItem>
                </MenuBarItem>
            </MenuBar>
        </Grid>
        <Grid Grid.Row="1" Padding="15">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto" />
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="200"/>
                <ColumnDefinition Width="Auto"/>
            </Grid.ColumnDefinitions>
            <TextBlock Grid.Column="0" x:Name="TBlock">You Clicked :</TextBlock>
            <TextBlock Grid.Column="1" x:Name="TBlock_2">State :</TextBlock>
        </Grid>
    </Grid>
</Window>
```

xaml.h
```
#pragma once
#include "MainWindow.g.h"

namespace winrt::App3::implementation
{
    struct MainWindow : MainWindowT<MainWindow>
    {
        MainWindow();

        int32_t MyProperty();
        void MyProperty(int32_t value);
        void MenuFlyoutItem_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void MenuFlyoutItem_Click_1(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void MenuFlyoutItem_Click_2(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void MenuFlyoutItem_Click_3(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void MenuFlyoutItem_Click_4(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void MenuFlyoutItem_Click_5(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void MenuFlyoutItem_Click_6(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void MenuFlyoutItem_Click_7(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void RadioMenuFlyoutItem_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void RadioMenuFlyoutItem_Click_1(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
    };
}

namespace winrt::App3::factory_implementation
{
    struct MainWindow : MainWindowT<MainWindow, implementation::MainWindow>
    {
    };
}
```

xaml.cpp
```
#include "pch.h"
#include "MainWindow.xaml.h"
#if __has_include("MainWindow.g.cpp")
#include "MainWindow.g.cpp"
#endif

using namespace winrt;
using namespace Microsoft::UI::Xaml;

// To learn more about WinUI, the WinUI project structure,
// and more about our project templates, see: http://aka.ms/winui-project-info.

namespace winrt::App3::implementation
{
    MainWindow::MainWindow()
    {
        InitializeComponent();
        this->Title(L"메뉴바 제작");
        TBlock_2().Text(L"State : ON");
    }

    int32_t MainWindow::MyProperty()
    {
        throw hresult_not_implemented();
    }

    void MainWindow::MyProperty(int32_t /* value */)
    {
        throw hresult_not_implemented();
    }

}

void winrt::App3::implementation::MainWindow::MenuFlyoutItem_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock().Text(L"You Clicked : Save");
}

void winrt::App3::implementation::MainWindow::MenuFlyoutItem_Click_1(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock().Text(L"You Clicked : Open");
}

void winrt::App3::implementation::MainWindow::MenuFlyoutItem_Click_2(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock().Text(L"You Clicked : Cut");
}

void winrt::App3::implementation::MainWindow::MenuFlyoutItem_Click_3(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock().Text(L"You Clicked : Copy");
}

void winrt::App3::implementation::MainWindow::MenuFlyoutItem_Click_4(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock().Text(L"You Clicked : Paste");
}

void winrt::App3::implementation::MainWindow::MenuFlyoutItem_Click_5(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    MessageBox(NULL, L"도움말", L"도움말", MB_OK);
}

void winrt::App3::implementation::MainWindow::MenuFlyoutItem_Click_6(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock().Text(L"You Clicked : Bluetooth");
}

void winrt::App3::implementation::MainWindow::MenuFlyoutItem_Click_7(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock().Text(L"You Clicked : Email");
}

void winrt::App3::implementation::MainWindow::RadioMenuFlyoutItem_Click(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock_2().Text(L"State : ON");
}

void winrt::App3::implementation::MainWindow::RadioMenuFlyoutItem_Click_1(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e)
{
    TBlock_2().Text(L"State : OFF");
}
```
