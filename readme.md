I den här labben kommer vi att utforska MVC i .NET Core

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
4. Skapa en mapp Views i projekt-roten, undermapp `Home` med en ny fil `Index.cshtml`.
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
11. Byt ut `app.UseEndpoints``-anropet i StartUp.Configure mot
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
4. Kompilera och kör så borde du se "© Talangprogrammet 2021"
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
Labb 6 - Section, partial views, paginering
----------------------------------------------------------------

1. Bryt ut renderingen av team-medlemmar i `Home`-vyn till en partial view som renderar ut en `Consultant`.
   Som namnkonvention börjar partiella vyer med `_` t.ex. `_Consultant.cshtml`.
2. Lägg till en sektion i din `_Layout.cshtml` (`@RenderSection`) och ange text i en vy som skrivs i denna sektion.
3. Hur skiljer sig sektioner från partiella vyer och vad har de för användningsområden?

----------------------------------------------------------------
The end
----------------------------------------------------------------

- Du var en snabb en! 🏎️💨 
- Hjälp din granne, gå tillbaks till gamla labbar eller lös lite uppgifter på Exercism: https://exercism.io/tracks/csharp/exercises
