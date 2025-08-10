# Avalonia.Windows.Scaling.Bug
Scaling/Resolution bug on Windows

Step to reproduce:
1. Create project with latest prerealse Avalonia
2. Change App.xaml.cs code
3. Run and change resolution or scaling

```
using Avalonia;
using Avalonia.Controls;
using Avalonia.Controls.ApplicationLifetimes;
using Avalonia.Markup.Xaml;
using Avalonia.Platform;
using Avalonia.Threading;

namespace Avalonia.Windows.Scaling.Bug;

public partial class App : Application
{
    public override void Initialize()
    {
        AvaloniaXamlLoader.Load(this);
    }

    public override void OnFrameworkInitializationCompleted()
    {
        if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
        {
            desktop.MainWindow = new MainWindow()
            {
                WindowState = WindowState.Maximized,
                ExtendClientAreaToDecorationsHint = false,
                ExtendClientAreaChromeHints = ExtendClientAreaChromeHints.NoChrome,
                ExtendClientAreaTitleBarHeightHint = -1,
                SystemDecorations = SystemDecorations.None
            };
        }

        base.OnFrameworkInitializationCompleted();
    }
}
```

After scaling down (from 150% to 100%):
<img width="3838" height="2158" alt="scaling down" src="https://github.com/user-attachments/assets/930b9c48-3c8a-4771-9204-6362f2c92ccd" />

After scaling up (from 150% to 175%) it acts as WindowState.Fullscreen

And same for changing screen resolution up and down.

Dirty and temporary fix:
```
desktop.MainWindow.ScalingChanged += (s, e) =>
                  {
                      desktop.MainWindow.WindowState = WindowState.Minimized;
                      Dispatcher.UIThread.InvokeAsync(() =>
                      {
                          desktop.MainWindow.WindowState = WindowState.Maximized; 
                          desktop.MainWindow.Focus();
                      });
                  };
```
