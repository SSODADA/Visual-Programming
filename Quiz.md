#  Q2_1 두 수를 더하는 프로그램

xmal
```
<?xml version="1.0" encoding="utf-8"?>
<Window
    x:Class="Q2_1.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Q2_1"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    Title="Q2_1">

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

#include "MainWindow.g.h"

namespace winrt::Q2_1::implementation
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
#include "MainWindow.xaml.h"
#if __has_include("MainWindow.g.cpp")
#include "MainWindow.g.cpp"
#endif
#include "pch.h"
#include <sstream>
#include <winrt/Microsoft.UI.Windowing.h>
#include <winrt/Windows.Graphics.h>

using namespace winrt;
using namespace Microsoft::UI::Xaml;

// To learn more about WinUI, the WinUI project structure,
// and more about our project templates, see: http://aka.ms/winui-project-info.

namespace winrt::Q2_1::implementation
{
    int32_t MainWindow::MyProperty()
    {
        throw hresult_not_implemented();
    }

    void MainWindow::MyProperty(int32_t /* value */)
    {
        throw hresult_not_implemented();
    }

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

            auto appWindow = this->AppWindow();
            appWindow.Resize({ (int32_t)num1, (int32_t)num2 });
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
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    Title="App1">
	<Grid RowDefinitions="Auto,*" Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
		<!-- Toolbar -->
		<Grid Grid.Row="0" Padding="8" ColumnDefinitions="Auto,*,Auto,Auto,Auto,Auto" VerticalAlignment="Center">
			<!-- 색상 -->
			<Button x:Name="ColorButton" Content="색상" Margin="0,0,8,0">
				<Button.Flyout>
					<Flyout x:Name="ColorFlyout">
						<StackPanel Spacing="8" Width="260">
							<ColorPicker x:Name="PenColorPicker" ColorChanged="PenColorPicker_ColorChanged"/>
							<Rectangle x:Name="ColorPreview" Height="24" RadiusX="6" RadiusY="6"/>
							<StackPanel Orientation="Horizontal" Spacing="8" HorizontalAlignment="Right">
								<Button x:Name="ApplyColorButton" Content="적용" Click="ApplyColorButton_Click"/>
								<Button Content="닫기" Click="CloseColorFlyout_Click"/>
							</StackPanel>
						</StackPanel>
					</Flyout>
				</Button.Flyout>
			</Button>

			<!-- 두께 -->
			<StackPanel Grid.Column="1" Orientation="Horizontal" Spacing="8">
				<TextBlock Text="두께" VerticalAlignment="Center"/>
				<Slider x:Name="ThicknessSlider" Minimum="2" Maximum="64" Value="2"
                Width="200" ValueChanged="ThicknessSlider_ValueChanged"/>
				<TextBlock x:Name="ThicknessValueText" VerticalAlignment="Center" Text="2.0 px"/>
			</StackPanel>

			<!-- 저장/불러오기/지우기 -->
			<Button Grid.Column="2" x:Name="SaveButton" Content="저장" Margin="8,0,0,0" Click="SaveButton_Click"/>
			<Button Grid.Column="3" x:Name="LoadButton" Content="불러오기" Margin="8,0,0,0" Click="LoadButton_Click"/>
			<Button Grid.Column="4" x:Name="ClearButton" Content="지우기" Margin="8,0,0,0" Click="ClearButton_Click"/>

		</Grid>

		<!-- Drawing Area -->
		<Canvas x:Name="DrawCanvas" Grid.Row="1" Background="White"
            PointerPressed="DrawCanvas_PointerPressed"
            PointerMoved="DrawCanvas_PointerMoved"
            PointerReleased="DrawCanvas_PointerReleased"/>
	</Grid>
</Window>
```

xmal.h
```
#pragma once
#include "MainWindow.g.h"

#include <vector>
#include <cmath>
#include <string_view>
#include <cstdint>

// WinUI / C++/WinRT
#include <winrt/Windows.UI.h>
#include <winrt/Windows.Foundation.h>
#include <winrt/Microsoft.UI.Xaml.Shapes.h>
#include <winrt/Microsoft.UI.Xaml.Input.h>
#include <winrt/Microsoft.UI.Xaml.Controls.Primitives.h>
#include <winrt/Microsoft.UI.Xaml.Media.h>
#include <winrt/Microsoft.UI.Xaml.Media.Imaging.h>
#include <winrt/Windows.Data.Json.h>
#include <winrt/Windows.Graphics.Imaging.h>
#include <winrt/Windows.Storage.Streams.h>

// File pickers HWND init
#ifndef NOMINMAX
#define NOMINMAX 1
#endif
#include <windows.h>
#include <Shobjidl_core.h>
#include <microsoft.ui.xaml.window.h>
#ifdef min
#undef min
#endif
#ifdef max
#undef max
#endif

namespace winrt::Q2_2::implementation
{
    struct StrokeData
    {
        winrt::Windows::UI::Color color{};
        double thickness{ 2.0 };
        std::vector<winrt::Windows::Foundation::Point> points{};
    };

    struct MainWindow : MainWindowT<MainWindow>
    {
        MainWindow();
        int32_t MyProperty();
        void MyProperty(int32_t value);

        // Drawing
        void DrawCanvas_PointerPressed(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void DrawCanvas_PointerMoved(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);
        void DrawCanvas_PointerReleased(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e);

        // Toolbar
        void PenColorPicker_ColorChanged(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::Controls::ColorChangedEventArgs const& e);
        void ApplyColorButton_Click(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void CloseColorFlyout_Click(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void ThicknessSlider_ValueChanged(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::Controls::Primitives::RangeBaseValueChangedEventArgs const& e);

        // File I/O
        winrt::fire_and_forget SaveButton_Click(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        winrt::fire_and_forget LoadButton_Click(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);
        void ClearButton_Click(winrt::Windows::Foundation::IInspectable const&,
            winrt::Microsoft::UI::Xaml::RoutedEventArgs const& e);

    private:
        // drawing state
        winrt::Microsoft::UI::Xaml::Shapes::Polyline m_currentStroke{ nullptr };
        StrokeData m_currentData{};
        bool m_isDrawing{ false };

        // pen state
        winrt::Windows::UI::Color m_currentColor{ 255, 0, 0, 0 };
        winrt::Windows::UI::Color m_pendingColor{ 255, 0, 0, 0 };
        double m_currentThickness{ 2.0 };

        // strokes model
        std::vector<StrokeData> m_strokes;

        // UI apartment capture
        winrt::apartment_context m_ui;

        // ui helpers
        void UpdateColorPreview();
        void RedrawFromModel();
        void InitializePickerWithWindow(winrt::Windows::Foundation::IInspectable const& picker);
        winrt::Windows::Foundation::IAsyncAction ShowErrorDialog(winrt::hstring const& message);

        // canvas -> MNIST tensor (1x1x28x28, float32) with advanced preprocessing
        winrt::Windows::Foundation::IAsyncOperation<bool>
            TryRenderToMnistTensor(std::vector<float>& outTensor28x28);
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
#include "MainWindow.xaml.h"
#if __has_include("MainWindow.g.cpp")
#include "MainWindow.g.cpp"
#endif

// WinUI
#include <winrt/Microsoft.UI.Xaml.Media.h>
#include <winrt/Microsoft.UI.Xaml.Shapes.h>
#include <winrt/Microsoft.UI.Xaml.Controls.h>
#include <winrt/Microsoft.UI.Xaml.Input.h>
#include <winrt/Microsoft.UI.Input.h>
#include <winrt/Microsoft.UI.Windowing.h>
#include <winrt/Microsoft.UI.Xaml.Media.Imaging.h>

#include <winrt/Windows.Data.Json.h>
#include <winrt/Windows.Storage.h>
#include <winrt/Windows.Storage.Pickers.h>
#include <winrt/Windows.Graphics.Imaging.h>
#include <winrt/Windows.Storage.Streams.h>

#include <windows.h>
#include <Shobjidl_core.h>
#include <microsoft.ui.xaml.window.h>

#include <sstream>
#include <algorithm>
#include <array>
#include <limits>
#include <numeric>
#include <cmath>

using namespace winrt;
using namespace Microsoft::UI::Xaml;
using namespace Microsoft::UI::Xaml::Controls;
using namespace Microsoft::UI::Xaml::Input;
using namespace Microsoft::UI::Xaml::Media;
using namespace Microsoft::UI::Xaml::Media::Imaging;

using Windows::Data::Json::JsonArray;
using Windows::Data::Json::JsonObject;
using Windows::Data::Json::JsonValue;
using Windows::Foundation::IAsyncAction;

namespace winrt::Q2_2::implementation
{
    MainWindow::MainWindow()
    {
        InitializeComponent();
        m_ui = winrt::apartment_context{}; // UI apartment capture

        if (auto aw = this->AppWindow())
            aw.Resize({ 1024, 800 });

        if (auto cp = this->PenColorPicker())
            cp.Color(m_currentColor);

        UpdateColorPreview();

        if (auto tb = this->ThicknessValueText())
            tb.Text(L"2.0 px");
    }
    int32_t MainWindow::MyProperty() { throw hresult_not_implemented(); }
    void MainWindow::MyProperty(int32_t) { throw hresult_not_implemented(); }

    void MainWindow::InitializePickerWithWindow(Windows::Foundation::IInspectable const& picker)
    {
        if (auto native = this->try_as<::IWindowNative>())
        {
            HWND hwnd{};
            if (SUCCEEDED(native->get_WindowHandle(&hwnd)) && hwnd)
            {
                if (auto init = picker.try_as<::IInitializeWithWindow>())
                    (void)init->Initialize(hwnd);
            }
        }
    }

    IAsyncAction MainWindow::ShowErrorDialog(hstring const& message)
    {
        co_await m_ui; // UI thread
        ContentDialog dlg{};
        dlg.XamlRoot(this->Content().XamlRoot());
        dlg.Title(box_value(L"오류"));
        dlg.Content(box_value(message));
        dlg.CloseButtonText(L"확인");
        co_await dlg.ShowAsync();
    }

    void MainWindow::PenColorPicker_ColorChanged(IInspectable const&,
        Microsoft::UI::Xaml::Controls::ColorChangedEventArgs const& e)
    {
        m_pendingColor = e.NewColor();
        UpdateColorPreview();
    }

    void MainWindow::ApplyColorButton_Click(IInspectable const&, RoutedEventArgs const&)
    {
        m_currentColor = m_pendingColor;
        if (auto cp = this->PenColorPicker()) cp.Color(m_currentColor);
        UpdateColorPreview();
        if (auto f = this->ColorFlyout()) f.Hide();
    }

    void MainWindow::CloseColorFlyout_Click(IInspectable const&, RoutedEventArgs const&)
    {
        if (auto f = this->ColorFlyout()) f.Hide();
    }

    void MainWindow::ThicknessSlider_ValueChanged(IInspectable const&,
        Microsoft::UI::Xaml::Controls::Primitives::RangeBaseValueChangedEventArgs const& e)
    {
        m_currentThickness = e.NewValue();
        if (auto tb = this->ThicknessValueText())
        {
            wchar_t buf[32]{};
            swprintf_s(buf, L"%.1f px", m_currentThickness);
            tb.Text(buf);
        }
        if (m_currentStroke) m_currentStroke.StrokeThickness(m_currentThickness);
    }

    void MainWindow::DrawCanvas_PointerPressed(IInspectable const&, PointerRoutedEventArgs const& e)
    {
        auto canvas = this->DrawCanvas();
        if (!canvas) return;

        auto point = e.GetCurrentPoint(canvas);
        if (!point.Properties().IsLeftButtonPressed()) return;

        m_isDrawing = true;

        Microsoft::UI::Xaml::Shapes::Polyline pl;
        pl.Stroke(SolidColorBrush(m_currentColor));
        pl.StrokeThickness(m_currentThickness);
        pl.StrokeLineJoin(PenLineJoin::Round);
        pl.StrokeStartLineCap(PenLineCap::Round);
        pl.StrokeEndLineCap(PenLineCap::Round);

        auto pos = point.Position();
        Windows::Foundation::Point p{ static_cast<float>(pos.X), static_cast<float>(pos.Y) };
        pl.Points().Append(p);

        canvas.Children().Append(pl);
        canvas.CapturePointer(e.Pointer());

        m_currentStroke = pl;
        m_currentData = {};
        m_currentData.color = m_currentColor;
        m_currentData.thickness = m_currentThickness;
        m_currentData.points.push_back(p);

        e.Handled(true);
    }

    void MainWindow::DrawCanvas_PointerMoved(IInspectable const&, PointerRoutedEventArgs const& e)
    {
        if (!m_isDrawing || !m_currentStroke) return;

        auto canvas = this->DrawCanvas();
        if (!canvas) return;

        auto point = e.GetCurrentPoint(canvas);
        if (!point.IsInContact()) return;

        auto pos = point.Position();
        Windows::Foundation::Point p{ static_cast<float>(pos.X), static_cast<float>(pos.Y) };
        m_currentStroke.Points().Append(p);
        m_currentData.points.push_back(p);

        e.Handled(true);
    }

    void MainWindow::DrawCanvas_PointerReleased(IInspectable const&, PointerRoutedEventArgs const& e)
    {
        if (!m_isDrawing) return;

        if (auto canvas = this->DrawCanvas())
            canvas.ReleasePointerCaptures();

        m_isDrawing = false;

        if (!m_currentData.points.empty())
            m_strokes.push_back(std::move(m_currentData));

        m_currentStroke = nullptr;
        e.Handled(true);
    }

    winrt::fire_and_forget MainWindow::SaveButton_Click(IInspectable const&, RoutedEventArgs const&)
    {
        hstring err;
        try
        {
            JsonArray rootArray;
            for (auto const& s : m_strokes)
            {
                JsonObject o;

                JsonArray c;
                c.Append(JsonValue::CreateNumberValue(s.color.A));
                c.Append(JsonValue::CreateNumberValue(s.color.R));
                c.Append(JsonValue::CreateNumberValue(s.color.G));
                c.Append(JsonValue::CreateNumberValue(s.color.B));
                o.Insert(L"color", c);

                o.Insert(L"thickness", JsonValue::CreateNumberValue(s.thickness));

                JsonArray pts;
                for (auto const& pt : s.points)
                {
                    JsonObject jp;
                    jp.Insert(L"x", JsonValue::CreateNumberValue(pt.X));
                    jp.Insert(L"y", JsonValue::CreateNumberValue(pt.Y));
                    pts.Append(jp);
                }
                o.Insert(L"points", pts);

                rootArray.Append(o);
            }

            JsonObject doc;
            doc.Insert(L"version", JsonValue::CreateNumberValue(1));
            doc.Insert(L"strokes", rootArray);

            Windows::Storage::Pickers::FileSavePicker picker;
            InitializePickerWithWindow(picker);
            picker.SuggestedStartLocation(Windows::Storage::Pickers::PickerLocationId::DocumentsLibrary);
            picker.SuggestedFileName(L"strokes");
            picker.FileTypeChoices().Insert(L"JSON", single_threaded_vector<hstring>({ L".json" }));

            Windows::Storage::StorageFile file = co_await picker.PickSaveFileAsync();
            if (!file) co_return;

            co_await Windows::Storage::FileIO::WriteTextAsync(file, doc.Stringify());
        }
        catch (winrt::hresult_error const& ex)
        {
            hstring msg = ex.message();
            err = L"파일 저장 중 예외가 발생했습니다.\n\n" + msg;
        }
        catch (...)
        {
            err = L"파일 저장 중 알 수 없는 오류가 발생했습니다.";
        }

        if (!err.empty()) co_await ShowErrorDialog(err);
        co_return;
    }

    winrt::fire_and_forget MainWindow::LoadButton_Click(IInspectable const&, RoutedEventArgs const&)
    {
        hstring err;
        try
        {
            Windows::Storage::Pickers::FileOpenPicker picker;
            InitializePickerWithWindow(picker);
            picker.SuggestedStartLocation(Windows::Storage::Pickers::PickerLocationId::DocumentsLibrary);
            picker.FileTypeFilter().Append(L".json");

            Windows::Storage::StorageFile file = co_await picker.PickSingleFileAsync();
            if (!file) co_return;

            auto text = co_await Windows::Storage::FileIO::ReadTextAsync(file);

            JsonObject doc;
            if (!JsonObject::TryParse(text, doc)) { err = L"JSON 파싱 실패: 올바른 형식인지 확인하세요."; }
            else
            {
                if (!doc.HasKey(L"strokes")) { err = L"JSON에 'strokes' 키가 없습니다."; }
                else if (doc.Lookup(L"strokes").ValueType() != Windows::Data::Json::JsonValueType::Array) { err = L"'strokes' 값이 배열이 아닙니다."; }
                else
                {
                    auto arr = doc.Lookup(L"strokes").GetArray();
                    m_strokes.clear();

                    for (uint32_t i = 0; i < arr.Size(); ++i)
                    {
                        auto itemVal = arr.GetAt(i);
                        if (itemVal.ValueType() != Windows::Data::Json::JsonValueType::Object) continue;

                        auto o = itemVal.GetObject();
                        StrokeData s{};

                        if (o.HasKey(L"color") && o.Lookup(L"color").ValueType() == Windows::Data::Json::JsonValueType::Array)
                        {
                            auto c = o.Lookup(L"color").GetArray();
                            if (c.Size() == 4)
                            {
                                s.color = {
                                    static_cast<uint8_t>(c.GetAt(0).GetNumber()),
                                    static_cast<uint8_t>(c.GetAt(1).GetNumber()),
                                    static_cast<uint8_t>(c.GetAt(2).GetNumber()),
                                    static_cast<uint8_t>(c.GetAt(3).GetNumber())
                                };
                            }
                        }

                        if (o.HasKey(L"thickness") && o.Lookup(L"thickness").ValueType() == Windows::Data::Json::JsonValueType::Number)
                            s.thickness = o.Lookup(L"thickness").GetNumber();

                        if (o.HasKey(L"points") && o.Lookup(L"points").ValueType() == Windows::Data::Json::JsonValueType::Array)
                        {
                            auto pts = o.Lookup(L"points").GetArray();
                            s.points.reserve(pts.Size());
                            for (uint32_t k = 0; k < pts.Size(); ++k)
                            {
                                auto pv = pts.GetAt(k);
                                if (pv.ValueType() != Windows::Data::Json::JsonValueType::Object) continue;
                                auto jp = pv.GetObject();
                                if (jp.HasKey(L"x") && jp.HasKey(L"y") &&
                                    jp.Lookup(L"x").ValueType() == Windows::Data::Json::JsonValueType::Number &&
                                    jp.Lookup(L"y").ValueType() == Windows::Data::Json::JsonValueType::Number)
                                {
                                    float x = static_cast<float>(jp.Lookup(L"x").GetNumber());
                                    float y = static_cast<float>(jp.Lookup(L"y").GetNumber());
                                    s.points.push_back({ x, y });
                                }
                            }
                        }
                        if (!s.points.empty()) m_strokes.push_back(std::move(s));
                    }

                    co_await m_ui; // UI
                    RedrawFromModel();
                }
            }
        }
        catch (winrt::hresult_error const& ex)
        {
            hstring msg = ex.message();
            err = L"파일 읽기 중 예외가 발생했습니다。\n\n" + msg;
        }
        catch (...)
        {
            err = L"파일 읽기 중 알 수 없는 오류가 발생했습니다.";
        }

        if (!err.empty()) co_await ShowErrorDialog(err);
        co_return;
    }

    void MainWindow::UpdateColorPreview()
    {
        if (auto chip = this->ColorPreview())
            chip.Fill(SolidColorBrush(m_pendingColor));
    }

    void MainWindow::RedrawFromModel()
    {
        auto canvas = this->DrawCanvas();
        if (!canvas) return;

        canvas.Children().Clear();

        for (auto const& s : m_strokes)
        {
            if (s.points.empty()) continue;

            Microsoft::UI::Xaml::Shapes::Polyline pl;
            pl.Stroke(SolidColorBrush(s.color));
            pl.StrokeThickness(s.thickness);
            pl.StrokeLineJoin(PenLineJoin::Round);
            pl.StrokeStartLineCap(PenLineCap::Round);
            pl.StrokeEndLineCap(PenLineCap::Round);

            for (auto const& pt : s.points) pl.Points().Append(pt);

            canvas.Children().Append(pl);
        }
    }

    void MainWindow::ClearButton_Click(IInspectable const&, RoutedEventArgs const&)
    {
        if (auto canvas = this->DrawCanvas())
        {
            canvas.ReleasePointerCaptures();
            canvas.Children().Clear();
        }
        m_isDrawing = false;
        m_currentStroke = nullptr;
        m_currentData = {};
        m_strokes.clear();
    }
}
```
