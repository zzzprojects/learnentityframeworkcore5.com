# Xamarin

**Xamarin**.**Forms** is an open-source mobile UI framework from Microsoft for building iOS, Android, & Windows **apps** with .NET from a single shared codebase.

* Xamarin.Forms is a feature of Xamarin, the popular mobile development framework that extends the .NET developer platform with tools and libraries for building mobile apps.
* Use Xamarin.Forms built-in pages, layouts, and controls to build and design mobile apps from a single API that is highly extensible. Subclass any control to customize their behavior or define your own controls, layouts, pages, and cells to make your app pixel perfect.

### Create a Xamarin App

To start, we will create a Xamarin project. The project type comes with all template files to create Xamarin application before adding anything. Let's open Visual Studio 2019,  if you haven't already installed Visual Studio, go to the [Visual Studio downloads](https://visualstudio.microsoft.com/downloads) page to install it for free.

![](../.gitbook/assets/image%20%2830%29.png)

On the start window, choose **Create a new project**.

![](../.gitbook/assets/image%20%2838%29.png)

On the **Create a new project** window, enter or type _xamarin.forms_ in the search box. Next, choose **C\#** from the Language list. Select the  **Mobile App \(Xamarin.Forms\)** template, and then choose **Next**.

![](../.gitbook/assets/image%20%2847%29.png)

In the **Configure your new project** window, type or enter _**EFCore5InXamarinApp**_ in the **Project name** box and click on the **Create** button. 

![](../.gitbook/assets/image%20%2824%29.png)

On the **New Mobile App** page, select the **Flyout** option, check the **Andriod** checkbox, and click on the **Create** button.

![](../.gitbook/assets/image%20%2843%29.png)

Visual Studio opens your new project and includes the default code files in your project as shown in the **Solution Explorer**.

### Install Entity Framework Core

To use Entity Framework Core we need to install [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/) library. [I](https://www.nuget.org/packages/Z.EntityFramework.Extensions.EFCore/)t is available as a nuget package and you can install it using **Nuget Package Manager**.

In the **Package Manager Console** window, enter the following command.

```bash
PM> Install-Package Microsoft.EntityFrameworkCore
```

For SQLite, we need to install [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite) and will get all the packages required for EF Core.

```bash
PM> Install-Package Microsoft.EntityFrameworkCore.Sqlite
```

### Create a Data Model and Database Context

In **Solution Explorer**, right-click on the _**Models**_ folder and choose **Add &gt; Class**. Enter a class file name **Author.cs** and add the following code.

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using System.Text;

namespace EFCore5InXamarinApp.Models
{
    public class Author
    {
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public string Id { get; set; }
        public string Name { get; set; }
        public string Address { get; set; }
    }
}

```

To add a context class, right-click on the _**Services**_ folder in **Solution Explorer**, and choose **Add &gt; Class**. Enter a class file name **AuthorContext.cs** and add the following code.

```csharp
using EFCore5InXamarinApp.Models;
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Text;

namespace EFCore5InXamarinApp.Services
{
    class AuthorContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            var path = System.Environment.GetFolderPath(System.Environment.SpecialFolder.Personal);
            optionsBuilder.UseSqlite($"Data Source={path}/AuthorContext.db;");
        }

        public DbSet<Author> Authors { get; set; }
    }
}

```

Add another class to the _**Services**_ folder and name it **AuthorDataStore.cs** and add the following code.

```csharp
using EFCore5InXamarinApp.Models;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace EFCore5InXamarinApp.Services
{
    public class AuthorDataStore : IDataStore<Author>
    {
        AuthorContext _context;

        public AuthorDataStore()
        {
            _context = new AuthorContext();
        }

        public async Task<bool> AddItemAsync(Author author)
        {
            _context.Authors.Add(author);
            _context.SaveChanges();

            return await Task.FromResult(true);
        }

        public async Task<bool> UpdateItemAsync(Author author)
        {
            var oldAuthor = _context.Authors.Where((Author arg) => arg.Id == author.Id).FirstOrDefault();
            _context.Authors.Remove(oldAuthor);
            _context.Authors.Add(author);
            _context.SaveChanges();

            return await Task.FromResult(true);
        }

        public async Task<bool> DeleteItemAsync(string id)
        {
            var oldAuthor = _context.Authors.Where((Author arg) => arg.Id == id).FirstOrDefault();
            _context.Authors.Remove(oldAuthor);
            _context.SaveChanges();

            return await Task.FromResult(true);
        }

        public async Task<Author> GetItemAsync(string id)
        {
            return await Task.FromResult(_context.Authors.FirstOrDefault(s => s.Id == id));
        }

        public async Task<IEnumerable<Author>> GetItemsAsync(bool forceRefresh = false)
        {
            return await Task.FromResult(_context.Authors);
        }
    }
}
```

It implements the `IDataStore` interface and contains all the required database operations. To get the implementation of `IDataStore` add the following code the `BaseViewModel`.

```csharp
public IDataStore<Author> AuthDataStore => DependencyService.Get<IDataStore<Author>>();
```

### Add Views and ViewModels

#### Create New Author View and ViewModel

To create a new author view, right-click on the **Views** folder and select **Add &gt; New Item...** 

![](../.gitbook/assets/image%20%2829%29.png)

Select the **Content Page** template, enter `NewAuthorPage.xaml` in the **Name** field and click on the **Add** button. Replace the following code in `NewAuthorPage.xaml` file.

```csharp
<?xml version="1.0" encoding="UTF-8"?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="EFCore5InXamarinApp.Views.NewAuthorPage"
             Shell.PresentationMode="ModalAnimated"
             Title="New Author"
             xmlns:ios="clr-namespace:Xamarin.Forms.PlatformConfiguration.iOSSpecific;assembly=Xamarin.Forms.Core"
             ios:Page.UseSafeArea="true">
    <ContentPage.Content>
        <StackLayout Spacing="3" Padding="15">
            <Label Text="Name" FontSize="Medium" />
            <Entry Text="{Binding Name, Mode=TwoWay}" FontSize="Medium" />
            <Label Text="Address" FontSize="Medium" />
            <Editor Text="{Binding Address, Mode=TwoWay}" AutoSize="TextChanges" FontSize="Medium" Margin="0" />
            <StackLayout Orientation="Horizontal">
                <Button Text="Cancel" Command="{Binding CancelCommand}" HorizontalOptions="FillAndExpand"></Button>
                <Button Text="Save" Command="{Binding SaveCommand}" HorizontalOptions="FillAndExpand"></Button>
            </StackLayout>
        </StackLayout>
    </ContentPage.Content>

</ContentPage>
```

Now let's add a view model class for this page by adding a new class in the **ViewModels** folder, name it `NewAuthorViewModel.cs` and replace the following code.

```csharp
using EFCore5InXamarinApp.Models;
using System;
using System.Collections.Generic;
using System.Text;
using System.Windows.Input;
using Xamarin.Forms;

namespace EFCore5InXamarinApp.ViewModels
{
    public class NewAuthorViewModel : BaseViewModel
    {
        private string name;
        private string address;

        public NewAuthorViewModel()
        {
            SaveCommand = new Command(OnSave, ValidateSave);
            CancelCommand = new Command(OnCancel);
            this.PropertyChanged +=
                (_, __) => SaveCommand.ChangeCanExecute();
        }

        private bool ValidateSave()
        {
            return !String.IsNullOrWhiteSpace(name)
                && !String.IsNullOrWhiteSpace(address);
        }

        public string Name
        {
            get => name;
            set => SetProperty(ref name, value);
        }

        public string Address
        {
            get => address;
            set => SetProperty(ref address, value);
        }

        public Command SaveCommand { get; }
        public Command CancelCommand { get; }

        private async void OnCancel()
        {
            // This will pop the current page off the navigation stack
            await Shell.Current.GoToAsync("..");
        }

        private async void OnSave()
        {
            Author newAuthor = new Author()
            {
                Id = Guid.NewGuid().ToString(),
                Name = Name,
                Address = Address
            };

            await AuthDataStore.AddItemAsync(newAuthor);

            // This will pop the current page off the navigation stack
            await Shell.Current.GoToAsync("..");
        }
    }
}

```

Update the `NewAuthorPage.xaml.cs` to bind the view model with a view.

```csharp
using EFCore5InXamarinApp.Models;
using EFCore5InXamarinApp.ViewModels;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using Xamarin.Forms;
using Xamarin.Forms.Xaml;

namespace EFCore5InXamarinApp.Views
{
    public partial class NewAuthorPage : ContentPage
    {
        public Author Author { get; set; }
        public NewAuthorPage()
        {
            InitializeComponent();
            BindingContext = new NewAuthorViewModel();
        }
    }
}
```

#### Create Author Detail View and ViewModel

To create an author detail view, right-click on the **Views** folder and select **Add &gt; New Item...** 

![](../.gitbook/assets/image%20%2832%29.png)

Select the **Content Page** template, enter `AuthorDetailPage.xaml` in the **Name** field and click on the **Add** button. Replace the following code in `AuthorDetailPage.xaml` file.

```csharp
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="EFCore5InXamarinApp.Views.AuthorDetailPage"
             Title="{Binding Title}">

    <StackLayout Spacing="20" Padding="15">
        <Label Text="Name:" FontSize="Medium" />
        <Label Text="{Binding Name}" FontSize="Small"/>
        <Label Text="Address:" FontSize="Medium" />
        <Label Text="{Binding Address}" FontSize="Small"/>
    </StackLayout>

</ContentPage>
```

Now let's add a view model class for this page by adding a new class in the **ViewModels** folder, name it `AuthorDetailViewModel.cs` and replace the following code.

```csharp
using EFCore5InXamarinApp.Models;
using EFCore5InXamarinApp.Services;
using System;
using System.Diagnostics;
using System.Threading.Tasks;
using Xamarin.Forms;

namespace EFCore5InXamarinApp.ViewModels
{
    [QueryProperty(nameof(AuthorId), nameof(AuthorId))]
    public class AuthorDetailViewModel : BaseViewModel
    {
        private string authorId;
        private string name;
        private string address;
        public string Id { get; set; }

        public string Name
        {
            get => name;
            set => SetProperty(ref name, value);
        }

        public string Address
        {
            get => address;
            set => SetProperty(ref address, value);
        }

        public string AuthorId
        {
            get
            {
                return authorId;
            }
            set
            {
                authorId = value;
                LoadAuthorId(value);
            }
        }

        public async void LoadAuthorId(string authorId)
        {
            try
            {
                var author = await AuthDataStore.GetItemAsync(authorId);
                Id = author.Id;
                Name = author.Name;
                Address = author.Address;
            }
            catch (Exception)
            {
                Debug.WriteLine("Failed to Load Author");
            }
        }
    }
}

```

Update the `AuthorDetailPage.xaml.cs` to bind the view model with a view.

```csharp
using EFCore5InXamarinApp.ViewModels;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using Xamarin.Forms;
using Xamarin.Forms.Xaml;

namespace EFCore5InXamarinApp.Views
{
    [XamlCompilation(XamlCompilationOptions.Compile)]
    public partial class AuthorDetailPage : ContentPage
    {
        public AuthorDetailPage()
        {
            InitializeComponent();
            BindingContext = new AuthorDetailViewModel();
        }
    }
}
```

#### Create Authors View and ViewModel

To display all the authors from the database, right-click on the **Views** folder and select **Add &gt; New Item...** 

![](../.gitbook/assets/image%20%2834%29.png)

Select the **Content Page** template, enter `AuthorsPage.xaml` in the **Name** field and click on the **Add** button. Replace the following code in `AuthorsPage.xaml` file.

```markup
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="EFCore5InXamarinApp.Views.AuthorsPage"
             Title="{Binding Title}"
             xmlns:local="clr-namespace:EFCore5InXamarinApp.ViewModels"  
             xmlns:model="clr-namespace:EFCore5InXamarinApp.Models"  
             x:Name="BrowseItemsPage">

    <ContentPage.ToolbarItems>
        <ToolbarItem Text="Add" Command="{Binding AddAuthorCommand}" />
    </ContentPage.ToolbarItems>
    <!--
      x:DataType enables compiled bindings for better performance and compile time validation of binding expressions.
      https://docs.microsoft.com/xamarin/xamarin-forms/app-fundamentals/data-binding/compiled-bindings
    -->
    <RefreshView x:DataType="local:AuthorsViewModel" Command="{Binding LoadAuthorsCommand}" IsRefreshing="{Binding IsBusy, Mode=TwoWay}">
        <CollectionView x:Name="AuthorsListView"
                ItemsSource="{Binding Authors}"
                SelectionMode="None">
            <CollectionView.ItemTemplate>
                <DataTemplate>
                    <StackLayout Padding="10" x:DataType="model:Author">
                        <Label Text="{Binding Name}" 
                            LineBreakMode="NoWrap" 
                            Style="{DynamicResource ListAuthorTextStyle}" 
                            FontSize="16" />
                        <Label Text="{Binding Address}" 
                            LineBreakMode="NoWrap"
                            Style="{DynamicResource ListAuthorDetailTextStyle}"
                            FontSize="13" />
                        <StackLayout.GestureRecognizers>
                            <TapGestureRecognizer 
                                NumberOfTapsRequired="1"
                                Command="{Binding Source={RelativeSource AncestorType={x:Type local:AuthorsViewModel}}, Path=AuthorTapped}"		
                                CommandParameter="{Binding .}">
                            </TapGestureRecognizer>
                        </StackLayout.GestureRecognizers>
                    </StackLayout>
                </DataTemplate>
            </CollectionView.ItemTemplate>
        </CollectionView>
    </RefreshView>
</ContentPage>

```

Now let's add a view model class for this page by adding a new class in the **ViewModels** folder, name it `AuthorsViewModel.cs` and replace the following code.

```csharp
using EFCore5InXamarinApp.Models;
using EFCore5InXamarinApp.Views;
using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Diagnostics;
using System.Text;
using System.Threading.Tasks;
using Xamarin.Forms;

namespace EFCore5InXamarinApp.ViewModels
{
    public class AuthorsViewModel : BaseViewModel
    {
        private Author _selectedAuthor;

        public ObservableCollection<Author> Authors { get; }
        public Command LoadAuthorsCommand { get; }
        public Command AddAuthorCommand { get; }
        public Command<Author> AuthorTapped { get; }

        public AuthorsViewModel()
        {
            Title = "Authors";
            Authors = new ObservableCollection<Author>();
            LoadAuthorsCommand = new Command(async () => await ExecuteLoadAuthorCommand());

            AuthorTapped = new Command<Author>(OnAuthorSelected);

            AddAuthorCommand = new Command(OnAddAuthor);
        }

        async Task ExecuteLoadAuthorCommand()
        {
            IsBusy = true;

            try
            {
                Authors.Clear();
                var authors = await AuthDataStore.GetItemsAsync(true);
                foreach (var author in authors)
                {
                    Authors.Add(author);
                }
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex);
            }
            finally
            {
                IsBusy = false;
            }
        }

        public void OnAppearing()
        {
            IsBusy = true;
            SelectedAuthor = null;
        }

        public Author SelectedAuthor
        {
            get => _selectedAuthor;
            set
            {
                SetProperty(ref _selectedAuthor, value);
                OnAuthorSelected(value);
            }
        }

        private async void OnAddAuthor(object obj)
        {
            await Shell.Current.GoToAsync(nameof(NewAuthorPage));
        }

        async void OnAuthorSelected(Author author)
        {
            if (author == null)
                return;

            // This will push the AuthorDetailPage onto the navigation stack
            await Shell.Current.GoToAsync($"{nameof(AuthorDetailPage)}?{nameof(AuthorDetailViewModel.AuthorId)}={author.Id}");
        }
    }
}
```

Update the `AuthorsPage.xaml.cs` to bind the view model with a view.

```csharp
using EFCore5InXamarinApp.ViewModels;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using Xamarin.Forms;
using Xamarin.Forms.Xaml;

namespace EFCore5InXamarinApp.Views
{
    [XamlCompilation(XamlCompilationOptions.Compile)]
    public partial class AuthorsPage : ContentPage
    {
        AuthorsViewModel _viewModel;
        public AuthorsPage()
        {
            InitializeComponent();
            BindingContext = _viewModel = new AuthorsViewModel();
        }

        protected override void OnAppearing()
        {
            base.OnAppearing();
            _viewModel.OnAppearing();
        }
    }
}
```

Let's update the `App.xaml.cs` to create and initialize the database with test data.

```csharp
using EFCore5InXamarinApp.Models;
using EFCore5InXamarinApp.Services;
using EFCore5InXamarinApp.Views;
using System;
using System.Linq;
using Xamarin.Forms;
using Xamarin.Forms.Xaml;

namespace EFCore5InXamarinApp
{
    public partial class App : Application
    {

        public App()
        {
            InitializeComponent();

            using (var context = new AuthorContext())
            {
                context.Database.EnsureDeleted();
                context.Database.EnsureCreated();

                context.Authors.Add(new Author() { Name = "Karl Jablonski", Address = "Skagen 21, Stavanger, Norway" });
                context.Authors.Add(new Author() { Name = "Matti Karttunen", Address = "Keskuskatu 45, Helsinki, Finland" });
                context.Authors.Add(new Author() { Name = "Tom Erichsen", Address = "305 - 14th Ave. S. Suite 3B, Seattle, USA" });

                context.SaveChanges();

                var list = context.Authors.ToList();
            }

            DependencyService.Register<AuthorDataStore>();
            MainPage = new AppShell();
        }

        protected override void OnStart()
        {
        }

        protected override void OnSleep()
        {
        }

        protected override void OnResume()
        {
        }
    }
}

```

To register the route for the author pages, replace the following code in `AppShell.xaml.cs`

```csharp
using EFCore5InXamarinApp.ViewModels;
using EFCore5InXamarinApp.Views;
using System;
using System.Collections.Generic;
using Xamarin.Forms;

namespace EFCore5InXamarinApp
{
    public partial class AppShell : Xamarin.Forms.Shell
    {
        public AppShell()
        {
            InitializeComponent();
            Routing.RegisterRoute(nameof(AuthorDetailPage), typeof(AuthorDetailPage));
            Routing.RegisterRoute(nameof(NewAuthorPage), typeof(NewAuthorPage));
        }

        private async void OnMenuItemClicked(object sender, EventArgs e)
        {
            await Shell.Current.GoToAsync("//LoginPage");
        }
    }
}

```

Now we also need to add the options to the menu in `AppShell.xaml` file to navigate to the Authors page.

```csharp
<FlyoutItem Title="Authors" Icon="icon_feed.png">
    <ShellContent Route="AuthorsPage" ContentTemplate="{DataTemplate local:AuthorsPage}" />
</FlyoutItem>
```

Let's run your application and click on the Authors menu option.

![](../.gitbook/assets/image%20%2828%29.png)

To add a new author tap the **ADD** button which is on the top right corner, it will navigate to the **New Author Page**.

![](../.gitbook/assets/image%20%2827%29.png)

Enter **Name** and **Address** and click on the **SAVE** button and you will see a new author is added. 

![](../.gitbook/assets/image%20%2840%29.png)

To view the detail, tap on an author and it will navigate to the detail page.

![](../.gitbook/assets/image%20%2841%29.png)

