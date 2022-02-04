Scroll down for english instructions.

I den här labben kommer vi att utforska ASP.NET Core MVC.

----------------------------------------------------------------
Labb 0 - Förberedelser
----------------------------------------------------------------

1. Klona projektet med `git clone` till en lämplig mapp. Använd knappen "Clone" på Bitbucket och kopiera lämplig url. (Får du problem här kan du behöva lägga till en SSH-nyckel för din dator hos Gitlab, läs mer: https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/)

2. Öppna Mvc.sln med Visual Studio.

----------------------------------------------------------------
Labb 1 - Hello World!
----------------------------------------------------------------

1. Om du kör Visual Studio, under menyraden, välj konfiguration Debug, Any CPU, Mvc (inte IIS Express).
2. Kompilera och kör projektet (Start debugging, <kbd>F5</kbd> or <kbd>⌘</kbd>+<kbd>⏎</kbd>).
   Om du får problem med saknade nuget-paket: Högerklicka på lösningen i Solution Explorer och välj "Restore NuGet Packages"
3. Var i koden anges att "Hello World!" ska skrivas ut?
   Det här ska vi nu byta ut mot en riktigt html-vy.
4. Skapa en mapp `Views` i projekt-roten, undermapp `Home` med en ny fil `Index.cshtml`.
5. Filen ska innehålla `<p>Hello View!</p>`
6. Skapa en mapp `Controllers` och lägg till en klass `HomeController`.
7. Låt den nya klassen ärva från `Controller`för att göra det till en MVC controller
   (`using Microsoft.AspNetCore.Mvc`)
8. Lägg in en publik metod Index med returtyp `IActionResult` som returnerar vyn i steg 5 (med hjälp av `View()`).
   Hur vet projektet vilken vy den ska returnera?
9. I `StartUp.ConfigureServices`, lägg till `services.AddControllersWithViews();`.
10. Frivilligt men rekommenderat för att kunna göra ändringar i vyn utan att bygga om:
   Högerklicka på Solution 'Mvc' i Solution Explorer.
   Välj "Manage NuGet Packages for solution".
   Välj Browse.
   Sök efter `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`.
   Installera.
   I `StartUp.ConfigureServices` byt `services.AddControllersWithViews();` till `services.AddControllersWithViews().AddRazorRuntimeCompilation();`
11. Byt ut `app.UseEndpoints`-anropet i StartUp.Configure mot
```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapDefaultControllerRoute();
});
```
Vad innebär det här steget?

12. Kompilera och kör. Grattis, du har skapat en default route, en controller och en vy!


----------------------------------------------------------------
Labb 2 - Shared Layout
----------------------------------------------------------------

1. Skapa en ny undermapp till `Views` som du döper till `Shared`.
2. Kopiera över filen `_Layout.cshtml` till `Shared`-mappen.
3. Skapa en ny fil `Views/_ViewStart.cshtml` med `@{ Layout = "_Layout"; }`
4. Kompilera och kör så borde du se "© Talangprogrammet 2022"
5. Byt ut "Talangprogrammet" i _Layout.cshtml mot ditt eget namn och ladda om.<br>
    `_ViewStart` direkt under `Views` kommer anropas av konvention i MVC men hur hänger
    `_Layout` ihop med din vy?

----------------------------------------------------------------
Labb 3 - Model
----------------------------------------------------------------

1. Skapa en mapp i projekt-roten som du döper till `Models`.
2. Skapa en klass som du döper till `Project`.
3. Lägg till några properties till din modell:
```csharp
        public int Id { get; set; }
        public string Name { get; set; }
        public string ProductOwner { get; set; }
        public List<string> Team { get; set; }
        public DateTime StartDate { get; set; }
```

4. Gå till `HomeController` och ändra `Index`-metoden:
```csharp
        public ViewResult Index()
        {
            var project = new Project
            {
                Id = 1,
                Name = "Ett projekt",
                ProductOwner = "Ulf Sidemo",
                Team = new List<string>
                {
                    "Anna Andersson",
                    "Per Persson",
                    "Oskar Oskarsson"
                },
                StartDate = new DateTime(2019, 9, 2)
            };

            return View("Index", project.Name);
        }
```

5. Gå till `Views/Home/Index.cshtml` och ta in projekt-namnet genom att byta ut innehållet mot
```csharp
        @model string

        <h1>@Model</h1>
```

6. Kompilera och kör så borde du se "Ett projekt"

7. Just nu skickar vi bara titeln på projektet till vyn, men nu ska vi ändra det! 
   Ändra i controllern och vyn så vi skickar och tar emot hela Project-objektet.

8. Skriv ut produktägarens namn, startdatumet och alla team-medlemmar i vyn.

    Razor-syntax för att skriva C# i vyn är: `@{ statement }` och `@expression`.
    Se lite Razor-exempel här: https://www.w3schools.com/asp/razor_syntax.asp

    Tips: Du kan formattera datumet med t.ex. `.ToShortDateString()`

9. Skapa en ny modell `Consultant` med string property `Name` för att representera 
    team-medlemmar och använd den istället för string i `Team`-propertyn i `Project`-modellen.
    
    Du kan transformera namnlistan du redan har till en Consultant-lista mha LINQ:
    ```csharp
    Team = new List<string> {/* alla namn */}.Select(name => new Consultant { Name = name }).ToList();
    ```

10. Modifiera vyn så att data skrivs ut korrekt igen

----------------------------------------------------------------
Labb 4 - Dependency Injection
----------------------------------------------------------------

Nu ska vi bryta ut och separera delar av logiken som vi binder ihop med Dependency Injection.

1. Skapa en mapp Services i projekt-roten och lägg till en fil `ConsultantService.cs` med klassen ConsultantService
2. Skapa interfacet `IConsultantService` (skapa en ny fil eller lägg det i samma) med
   signaturen för en funktion `GetAllConsultants()` som returnerar en lista av `Consultant`.
3. Låt `ConsultantService` implementera ditt interface. Kopiera sen konsult-listan från `HomeController` och gör så att `GetAllConsultants()` returnerar den.
4. Sätt upp `ConsultantService` som en transient implementation av `IConsultantService`.
   Tips: Metoden `ConfigureServices` i `Startup.cs`.

   Vad innebär det? Vad är skillnaden på en transient och en singelton implementation?

5. Nu kan du lägga till `IConsultantService` som ett beroende till `HomeController`.
   Det gör du genom att lägga till `IConsultantService` som parameter i konsturktorn. 
   Om all konfiguration blivit rätt kommer dependency injection-ramverket nu skapa upp
   en ny instans av `ConsultantService` så fort den behövs av `HomeController`.
```csharp
        private readonly IConsultantService _consultantService;

        public HomeController(IConsultantService consultantService)
        {
            _consultantService = consultantService;
        }
```

6. Använd `_consultantService` för att hämta alla konsulter och populera team-listan.
7. Kompilera och kör så det ryker.

----------------------------------------------------------------
Labb 5 - Routing och Model binding
----------------------------------------------------------------

1. Skapa en ny controller `ConsultantController` med action `Index`, en vy
   `Views/Consultant/Index.cshtml` och skicka med en consultant från controllern
    till vyn som skriver ut konsulten. Testkör.

2. Skapa en action `Create` med `[HttpGet]`-attribut, och vy `Create.cshtml` med
    formulär innehållandes input-fält som matchar `Consultant`-modellen, antingen
    med vanlig html eller med helper-metoder i stil med
```csharp
@using (Html.BeginForm())
{
    @Html.TextBox("Name");
    <input type="submit" value="Skapa">
}
```

3. I din controller, skapa en action `Create` med `[HttpPost]`-attribut och en
   `Consultant` som argument (model binding). Returnera `Index`-vyn med denna.
4. Testkör och gå till `/consultant/create` skapa en ny konsult och verifiera att denna skickas vidare till
   Index-vyn. Varför har vi två `Create`-metoder, vad är skillnaderna och hur vet vi vilken som ska köras när?
7. Lägg till `int projectId` som ytterligare argument till din `Create`-action,
   och skapa ett input-fält i `Create.cshtml` för just `projectId`.
   Du behöver inte använda `projectId` till något i controllern eller modellen.
8. Kör i debug-läge (<kbd>F5</kbd> [Win], <kbd>⌘</kbd>+<kbd>⏎</kbd> [Mac]) med en breakpoint i 
   `Create-metoden` och kontrollera att du kan se både `Consultant` och `projectId` i `Create`. 

   Vad händer om du som användare skriver in något som inte är en int i formuläret, varför?

----------------------------------------------------------------
Labb 6 - Partial views & Sections
----------------------------------------------------------------

### Partial Views
1. Bryt ut rendreringen av team-medlemmar i `Index`-vyn till en partial view som renderar __en__ `Consultant`.
   Loopen måste alltså vara kvar i `Index`. Som namnkonvention börjar partiella vyer med `_` t.ex. `_Consultant.cshtml`.
2. Du måste också hitta ett sätt att "anropa" den partiella vyn från `Index`-vyn.
   Tips: `Html.RenderPartial` eller `await Html.RenderPartialAsync`.

### Sections
2. I `_Layout.cshtml`, lägg till referens till sektion (tips: `@RenderSection` t.ex. i footer).
3. I en vanlig vy, lägg till en sektion som skriver en text.
4. Kör. Nu ska texten synas på det ställe du angav i layout-vyn.

Hur skiljer sig sektioner från partiella vyer och vad har de för användningsområden?

----------------------------------------------------------------
Bonus Labb 7 - Razor Pages
----------------------------------------------------------------

1. Konvertera din MVC app till Razor Pages! Se https://docs.microsoft.com/en-us/aspnet/core/razor-pages/

----------------------------------------------------------------
The end
----------------------------------------------------------------

- Du var en snabb en! 🏎️💨 
- Hjälp din granne, gå tillbaks till gamla labbar eller lös lite uppgifter på Exercism: https://exercism.io/tracks/csharp/exercises

----------------------------------------------------------------
Lab 0 - Preparations
----------------------------------------------------------------

1. Clone the project with `git clone` to a suitable folder. Use the "Clone" button on Bitbucket and copy the appropriate url. (If you have problems here, you may need to add an SSH key for your computer at Gitlab, read more: https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/)

2. Open Mvc.sln with Visual Studio.

----------------------------------------------------------------
Lab 1 - Hello World!
----------------------------------------------------------------

1. If you are running Visual Studio, below the menu bar, select configuration Debug, Any CPU, Mvc (not IIS Express).
2. Compile and run the project (Start debugging, <kbd>F5</kbd> or <kbd>⌘</kbd> + <kbd>⏎</kbd>).
   If you have problems with missing nuget packages: Right-click on the solution in Solution Explorer and select "Restore NuGet Packages"
3. Where in the code is it stated that "Hello World!" should be printed?
   We will now replace this with a real html view.
4. Create a folder `Views` in the project root, subfolder `Home` with a new file `Index.cshtml`.
5. The file should contain `<p>Hello View!</p>`
6. Create a `Controllers` folder and add a` HomeController` class.
7. Let the new class inherit from `Controller` to make it an MVC controller
   (`using Microsoft.AspNetCore.Mvc`)
8. Enter a public method Index with return type `IActionResult` that returns the view in step 5 (using `View()`).
   How does the project know which view to return?
9. In `StartUp.ConfigureServices`, add `services.AddControllersWithViews();`.
10. Voluntary but recommended to be able to make changes to the view without rebuilding:
   Right-click on Solution 'Mvc' in Solution Explorer.
   Select "Manage NuGet Packages for solution".
   Select Browse.
   Search for `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`.
   Install.
   In `StartUp.ConfigureServices`, switch `services.AddControllersWithViews();` to `services.AddControllersWithViews().AddRazorRuntimeCompilation();`
11. Replace the `app.UseEndpoints` call in StartUp.Configure with
```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapDefaultControllerRoute();
});
```
What does this step mean?

12. Compile and run. Congratulations, you have created a default route, a controller and a view!

----------------------------------------------------------------
Lab 2 - Shared Layout
----------------------------------------------------------------

1. Create a new subfolder for `Views` which you name `Shared`.
2. Copy the file `_Layout.cshtml` to the `Shared` folder.
3. Create a new file `Views/_ViewStart.cshtml` with `@{ Layout = "_Layout"; }`
4. Compile and run and you should see "© Talent Program 2022"
5. Replace the "Talent Program" in _Layout.cshtml with your own name and reload.<br>
    `_ViewStart` directly under` Views` will be called by convention in MVC but how does
    `_Layout` hang along with your view?

----------------------------------------------------------------
Lab 3 - Model
----------------------------------------------------------------

1. Create a folder in the project root that you name `Models`.
2. Create a class that you name `Project`.
3. Add some properties to your model:
```csharp
        public int Id { get; set; }
        public string Name { get; set; }
        public string ProductOwner { get; set; }
        public List<string> Team { get; set; }
        public DateTime StartDate { get; set; }
```

4. Go to `HomeController` and change the `Index` method:
```csharp
        public ViewResult Index()
        {
            var project = new Project
            {
                Id = 1,
                Name = "Ett projekt",
                ProductOwner = "Ulf Sidemo",
                Team = new List<string>
                {
                    "Anna Andersson",
                    "Per Persson",
                    "Oskar Oskarsson"
                },
                StartDate = new DateTime(2019, 9, 2)
            };

            return View("Index", project.Name);
        }
```

5. Go to `Views/Home/Index.cshtml` and use the project name by changing the content to
```csharp
        @model string

        <h1>@Model</h1>
```

6. Compile and run and you should see "Ett projekt"

7. Right now we are just sending the title of the project to the view, but now we are going to change it!
   Change the controller and the view so that we send and receive the entire Project object.

8. Print the product owner's name, start date, and all team members in the view.

    Razor syntax for typing C# in the view is: `@{ statement }` and `@expression`.
    See some Razor examples here: https://www.w3schools.com/asp/razor_syntax.asp

    Tip: You can format the date with e.g. `.ToShortDateString()`

9. Create a new model `Consultant` with string property `Name` to represent
    team members and use it instead of string in the `Team` property in the `Project` model.
    
    You can transform the name list you already have into a Consultant list using LINQ:
    ```csharp
    Team = new List<string> {/* all names */}.Select(name => new Consultant {Name = name}).ToList();
    ```

10. Modify the view so that data is printed correctly again

----------------------------------------------------------------
Lab 4 - Dependency Injection
----------------------------------------------------------------

Now we will break out and separate parts of the logic that we connect with Dependency Injection.

1. Create a Services folder in the project root and add a file `ConsultantService.cs` with the ConsultantService class
2. Create the interface `IConsultantService` (create a new file or put it in the same) with
   the signature of a function `GetAllConsultants()` which returns a list of `Consultant`.
Let `ConsultantService` implement your interface. Then copy the consultant list from `HomeController` and have `GetAllConsultants()` return it.
4. Set up `ConsultantService` as a transient implementation of `IConsultantService`.
   Tip: The `ConfigureServices` method in `Startup.cs`.

   What does that mean? What is the difference between a transient and a singleton implementation?

5. You can now add `IConsultantService` as a dependency to `HomeController`.
   You do this by adding `IConsultantService` as a parameter in the construktor.
   If all configuration is correct, the dependency injection framework will now create
   a new instance of `ConsultantService` as soon as it is needed by `HomeController`.
```csharp
        private readonly IConsultantService _consultantService;

        public HomeController(IConsultantService consultantService)
        {
            _consultantService = consultantService;
        }
```

6. Use `_consultantService` to get all consultants and populate the team list.
7. Compile and run!

----------------------------------------------------------------
Lab 5 - Routing and Model binding
----------------------------------------------------------------

1. Create a new controller `ConsultantController` with action `Index`, a view
   `Views/Consultant/Index.cshtml` and send a consultant from the controller
    to the view that prints the consultant. Test run.

2. Create an action `Create` with `[HttpGet]` attribute, and view `Create.cshtml` with a
    form containing input fields that match the `Consultant` model, either
    with regular html or with helper methods along the lines of
```csharp
@using (Html.BeginForm())
{
    @Html.TextBox("Name");
    <input type="submit" value="Skapa">
}
```

3. In your controller, create an action `Create` with `[HttpPost]` attribute and one
   `Consultant` as an argument (model binding). Return the `Index` view with this.
4. Test run and go to `/consultant/create` create a new consultant and verify that this is forwarded to
   Index view. Why do we have two `Create` methods, what are the differences and how do we know which one to run when?
7. Add `int projectId` as an additional argument to your` Create` action,
   and create an input field in `Create.cshtml` for just `projectId`.
   You do not need to use `projectId` for anything in the controller or model.
8. Run in debug mode (<kbd>F5</kbd> [Win], <kbd>⌘</kbd> + <kbd>⏎</kbd> [Mac]) with a breakpoint in
   `Create method` and check that you can see both `Consultant` and `projectId` in `Create`.

   What happens if you as a user enter something that is not an int in the form, why?

----------------------------------------------------------------
Lab 6 - Partial views & Sections
----------------------------------------------------------------

### Partial Views
1. Extract the rendering of team members in the `Index` view to a partial view that renders __one__ `Consultant`.
   The loop must therefore remain in the `Index`. As a naming convention, partial views begin with `_` e.g. `_Consultant.cshtml`.
2. You must also find a way to "call" the partial view from the `Index` view.
   Tip: `Html.RenderPartial` or `await Html.RenderPartialAsync`.

### Sections
2. In `_Layout.cshtml`, add reference to section (hint: `@RenderSection` eg in footer).
3. In a standard view, add a section that writes a text.
4. Run. The text should now be visible in the place you specified in the layout view.

How do sections differ from partial views and what are their uses?

----------------------------------------------------------------
Bonus Lab 7 - Razor Pages
----------------------------------------------------------------

1. Convert your MVC app to Razor Pages! See https://docs.microsoft.com/en-us/aspnet/core/razor-pages/

----------------------------------------------------------------
The end
----------------------------------------------------------------

- You were a fast one! 🏎️💨 
- Help your neighbour, go back to old labs or solve some excercises on Exercism: https://exercism.io/tracks/csharp/exercises
