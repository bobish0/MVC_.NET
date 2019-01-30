I den här labben kommer vi att utforska MVC i .NET Core

----------------------------------------------------------------
Labb 0 - Förberedelser (Visual Studio Professional)
----------------------------------------------------------------

Se till att lägga in er ssh-nyckel här för att kunna klona projektet.
Hämta ner Mvc projektet från gitlab till din Projects mapp med:
$ git clone git@git.valtech.se:talangprogrammet/Mvc.git
Öppna Mvc.sln med Visual Studio

----------------------------------------------------------------
Labb 1 - Hello World!
----------------------------------------------------------------

1. Kompilera och kör projektet (Start without debugging, ctrl+F5 (Win), cmd+alt+enter (Mac))
    Om du får problem med saknade nuget-paket: 
        Högerklicka på solutionen i Solution Explorer och välj "Restore NuGet Packages"

2. Var i koden anges att "Hello World!" ska skrivas ut?
    Det här ska vi nu byta ut mot en riktigt html-vy

3. Skapa en mapp Views i projekt-roten, undermapp Home med en ny fil Index.cshtml
4. Filen ska innehålla "<p>Hello View!</p>"
5. Skapa en mapp Controllers och lägg till en klass HomeController
6. Låt den nya klassen ärva från "Controller" för att göra det till en MVC controller
    (using Microsoft.AspNetCore.Mvc)

7. Lägg in en publik metod Index med returtyp IActionResult som returnerar vyn i steg 5 mha View().
    Hur vet projektet vilken vy den ska returnera?

8. I Startup.cs: Lägg till "services.AddMvc();" i StartUp.ConfigureServices och importera
    "Microsoft.AspNetCore.Mvc".
9. Byt ut app.Run-anropet i StartUp.Configure mot

            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });

    Vad innebär det här steget?

10. Kompilera och kör (ctrl+F5 / cmd+alt+enter). Grattis, du har skapat en defaultroute, en controller och en vy!


----------------------------------------------------------------
Labb 2 - Shared Layout
----------------------------------------------------------------

1. Skapa en ny undermapp till Views som du döper till Shared
2. Kopiera in _Layout.cshtml till Shared-mappen
3. Skapa en ny fil Views/_ViewStart.cshtml med @{ Layout = "_Layout"; }
4. Kompilera och kör så borde du se "© Talangprogrammet 2019"
5. Byt ut "Talangprogrammet" i _Layout.cshtml mot ditt eget namn och ladda om
    _ViewStart direkt under Views kommer anropas by convention i MVC men hur hänger
    _Layout ihop med din vy?


----------------------------------------------------------------
Labb 3 - Model
----------------------------------------------------------------

1. Skapa en mapp i projekt-roten som du döper till Models
2. Skapa en klass som du döper till Project
3. Lägg till några properties till din modell:

        public int Id { get; set; }
        public string Name { get; set; }
        public string ProductOwner { get; set; }
        public List<string> Team { get; set; }
        public DateTime StartDate { get; set; }

4. Gå till HomeController och ändra Index-metoden:

        public ViewResult Index()
        {
            var project = new Project
            {
                Id = 1,
                Name = "Ett projekt",
                ProductOwner = "Ulf Sidemo",
                Team = new List<string>
                {
                    "Karin Melin", "Sebastian Lhådö", "David  Bragmark", "Tim Kulich",
                    "Josef Hansson Karakoca", "David Wajngot", "Johannes Almroth", "Adam Woods",
                    "Nurhussen Saleh", "Jesper Saxer", "Malin von Matern", "Anton Carlsson", 
                    "Jesper Svennebring", "Robin Rosberg"
                },
                StartDate = new DateTime(2019, 2, 6)
            };

            return View("Index", project.Name);
        }

5. Gå till Views/Home/Index.cshtml och ta in projekt-namnet genom att byta ut innehållet mot

        @model string

        <h1>@Model</h1>



6. Kompilera och kör så borde du se "Ett projekt"
7. Ändra vymodellen till Project (i vy och controller)
8. Skriv ut produktägarens namn, startdatumet och alla team-medlemmar i vyn
    Razor-syntax för att skriva C# i vyn är: @{ /* C#kod */ } och @C#Variabel
    Se lite Razor-exempel här: https://www.w3schools.com/asp/razor_syntax.asp
    Tips: Du kan formattera datumet med t.ex. .ToString("yyyy-MM-dd")

9. Skapa en ny modell Consultant med string property Name för att representera 
    team-medlemmar och använd den istället för string i Team-propertyn i Project-modellen.
    Du kan transformera namnlistan du redan har till en Consultant-lista mha LINQ:
    Team = new List<string> {...}.Select(name => new Consultant { Name = name }).ToList();

10. Modifiera vyn så att data skrivs ut korrekt igen


----------------------------------------------------------------
Labb 4 - Routing och Model binding
----------------------------------------------------------------

1. Skapa en ConsultantController med action Index, en vy
    Views/Consultant/Index.cshtml och skicka med en consultant från controllern
    till vyn som skriver ut konsulten. Testkör.

2. Skapa en action Create med [HttpGet]-attribut, och vy Create.cshtml med
    formulär innehållandes input-fält som matchar Consultant-modellen, antingen
    med vanlig html eller med helper-metoder i stil med

        @using (Html.BeginForm())
        {
            @Html.TextBox("Name");
            <input type="submit" value="Skapa">
        }


3. I din controller, skapa en action Create med [HttpPost]-attribut och en
    Consultant som argument (model binding). Returnera Index-vyn med denna.
4. Testkör och gå till /consultant/create skapa en ny konsult och verifiera att denna skickas vidare till
    Index-vyn. Varför har vi två Create-metoder, vad är skillnaderna och hur vet vi vilken som ska köras när?
7. Lägg till "int projectId" som ytterligare argument till din Create-action,
    och skapa ett input-fält i Create.cshtml
8. Debugga och kontrollera att du kan se både Consultant och projectId i Create. 
    Vad händer om du som användare skriver in något som inte är en int i formuläret?

----------------------------------------------------------------
Labb 5 - Section, partial views, paginering
----------------------------------------------------------------

1. Lägg till en section i din _Layout.cshtml (@RenderSection) och ange text i en vy som skrivs
    i denna section
2. Bryt ut renderingen av team-medlemmar i Home-vyn till en partial view som renderar ut en Consultant.
    Som namnkonvention börjar partiella vyer med '_' t.ex. _Consultant.cshtml
3. Hur skiljer sig sektioner från partiella vyer och vad har de för användningsområden?

----------------------------------------------------------------
Labb 6 - Dependency Injection
----------------------------------------------------------------

Nu ska vi bryta ut och separera delar av logiken som vi binder ihop med Dependency Injection.

1. Skapa en mapp Services i projekt-roten och lägg till en fil ConsultantService.cs med klassen ConsultantService
2. Skapa interfacet IConsultantService (skapa en ny fil eller lägg det i samma) med
    funktionen GetAllConsultants() som returnerar en lista av Consultant.
3. Låt ConsultantService implementera ditt interface
4. Kopiera konsult-listan från HomeController och gör så att GetAllConsultants() returnerar den.
5. I Startup.cs: sätt upp ConsultantService som en transient implementation av IConsultantService i metoden ConfigureServices.
    Vad innebär det? Vad är skillnaden på en transient och en singelton implementation?
6. Nu kan injecta IConsultantService i HomeController genom att ta den som constructorparameter:

        private readonly IConsultantService _consultantService;

        public HomeController(IConsultantService consultantService)
        {
            _consultantService = consultantService;
        }

7. Använd _consultantService för att hämta alla konsulter och populera team-listan.
8. Kompilera och kör så det ryker.

----------------------------------------------------------------
Labb 7 - The end
----------------------------------------------------------------

- Du var en snabb en! 🏎️💨 
- Hjälp din granne eller lös lite uppgifter på Exercism: https://exercism.io/tracks/csharp/exercises