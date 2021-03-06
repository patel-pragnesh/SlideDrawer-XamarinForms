# SlideDrawer-XamarinForms

Create a Custom View in Xamarin Forms by selecting ContentView in the template section.

Here is sample XAML design which can be altered according to your needs.

```
<ContentView x:Class="SampleAppTable.MyView"
             xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">
    <ContentView.Content>
            <StackLayout HeightRequest="600"
                         HorizontalOptions="FillAndExpand"
                         VerticalOptions="End"
                         BackgroundColor="Red"  />
    </ContentView.Content>
</ContentView>
```

On the code behind side, create few bindable properties where you can handle slides and easing functions.

```
public partial class MyView : ContentView
	{
		public MyView()
		{
			InitializeComponent();
		}

		public static readonly BindableProperty EasingFunctionProperty =
  			BindableProperty.Create<MyView, string>(p => p.EasingFunction, "SinIn", BindingMode.TwoWay, propertyChanged: OnEasingFunctionChanged);


		public static readonly BindableProperty DefaultHeightProperty =
			BindableProperty.Create<MyView, double>(p => p.DefaultHeight, 0, BindingMode.TwoWay, propertyChanged: DefaultHeightChanged);

		public static readonly BindableProperty IsSlideOpenProperty =
			BindableProperty.Create<MyView, bool>(p => p.IsSlideOpen, false, BindingMode.TwoWay, propertyChanged: SlideOpenClose);

		private Easing _easingFunction;
		public string EasingFunction {
			get { return (string)GetValue(EasingFunctionProperty); }
			set { SetValue(EasingFunctionProperty, value); }
		}

		public double DefaultHeight {
			get { return (double)GetValue(DefaultHeightProperty); }
			set { SetValue(DefaultHeightProperty, value); }
		}

		public bool IsSlideOpen {
			get { return (bool)GetValue(IsSlideOpenProperty); }
			set { SetValue(IsSlideOpenProperty, value); }
		}

		private static Easing GetEasing(string easingName)
		{
			switch (easingName) {
				case "BounceIn": return Easing.BounceIn;
				case "BounceOut": return Easing.BounceOut;
				case "CubicInOut": return Easing.CubicInOut;
				case "CubicOut": return Easing.CubicOut;
				case "Linear": return Easing.Linear;
				case "SinIn": return Easing.SinIn;
				case "SinInOut": return Easing.SinInOut;
				case "SinOut": return Easing.SinOut;
				case "SpringIn": return Easing.SpringIn;
				case "SpringOut": return Easing.SpringOut;
				default: throw new ArgumentException(easingName + " is not valid");
			}
		}

		private static void DefaultHeightChanged(BindableObject bindable, double oldValue, double newValue)
		{
			(bindable as MyView).IsVisible = false;
			(bindable as MyView).TranslationY = newValue;
		}

		private static void OnEasingFunctionChanged(BindableObject bindable, string oldvalue, string newvalue)
		{
			(bindable as MyView).EasingFunction = newvalue;
			(bindable as MyView)._easingFunction = GetEasing(newvalue);
		}

		private static async void SlideOpenClose(BindableObject bindable, bool oldValue, bool newValue)
		{
			if (newValue) {
				(bindable as MyView).IsVisible = true;
				await (bindable as MyView).TranslateTo(0, App.Current.MainPage.Height - 400, 250, Easing.SinInOut);
				newValue = false;
			} else {
				await (bindable as MyView).TranslateTo(0, App.Current.MainPage.Height, 250, Easing.SinInOut);
				(bindable as MyView).IsVisible = false;
				newValue = true;
			}
		}
	}
  ```
  
  Now Create a Sample Page where you can integrate the custom slide view. This sample is drived through normal MVVM 
  
  ```
  <ContentPage x:Class="SampleAppTable.MyPage"
             xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:controls="clr-namespace:SampleAppTable;assemnly=SampleAppTable"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">
    <ContentPage.Content>
        <StackLayout>
            <StackLayout.GestureRecognizers>
                <TapGestureRecognizer Command="{Binding TappedCommand}" />
            </StackLayout.GestureRecognizers>
            <Button HorizontalOptions="Center"
                    VerticalOptions="Center"
                    Command="{Binding SlideOpenCommand}"
                    Text="Open Menu" />
			<controls:MyView x:Name="MyLayout" DefaultHeight="{Binding DefaultHeight}" IsSlideOpen="{Binding IsSlide}" />
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```

Create a ViewModel and define the command and properties which are necessary.

```
[ImplementPropertyChanged]
	public class MainViewModel
	{
		public ICommand TappedCommand { get; set; }
		public ICommand SlideOpenCommand { get; set; }
		public double DefaultHeight { get; set; }

		public bool IsSlide { get; set; }

		public MainViewModel()
		{
			TappedCommand = new Command(CloseMenu);
			SlideOpenCommand = new Command(SlideOpen);
			DefaultHeight = App.Current.MainPage.Height;
			IsSlide = false;
		}

		private void CloseMenu()
		{
			IsSlide = false;
		}

		private void SlideOpen()
		{
			if (IsSlide) {
				IsSlide = false;
			} else {
				IsSlide = true;
			}
		}
	}
  ```
  
  I have used PropertyChanged package, in order to manage INotifyPropertyChanged automatically.
  
  In the MyPage.Xaml.Cs specify the binding context.
  
  ```
  protected override void OnAppearing()
		{
			base.OnAppearing();
			this.BindingContext = new MainViewModel();
		}
   ```
   
   Done.... 
   
   ![](https://github.com/guntidheerajkumar/SlideDrawer-XamarinForms/blob/master/Slide_Output.gif)
   

  
  
  
