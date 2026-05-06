# C#
using System.ComponentModel.DataAnnotations;

[Reqired(ErrorMessage = "Message")]

[MaxLength(200)]

[ReqularExpresssion(@"[a-z]"), ErrorMessage = "Messsage"]

[DisplayName("Nazev")]
## MVC

- Model - Data a logika... Třídy. Když je potřeba práce s daty, validace, počítání. Data z db, formuláře...
- View - HTML stránka (Razor .cshtml). Prezentace uživateli, co se vykresluje.
- Controller - Příjmá požadavky (http request), pracuje s modely, vrací view. Když je potřeba reagovat na URL, akci uživatele

#### Workflow

- Otevře URL (Product/Detail/1)
- Controller ProductController zacyhtí požadavek
- Načte data (Model)
- Pošle data do View
- Vykreslí HTML

#### připojení DB

- appsettings.json
    - "ConnectionStrings": {
    "DefaultConnection": "Data Source=rrs.db"
  }
- HomeController - Načte data a udělá list

    - **Dapper**
    ```
    using var connection = new SqliteConnection("Data Source=rss.db");
    var rss = connection.Query<Rss>("SELECT * FROM Rss").ToList();
    return View(rss);
    ```
- Models/Rss - Třída Rss se kterou se pracuje
- Views/Home/Index
    - ``@model List<Rss> //Nahoře``
    ```
    <table>
    @foreach (var item in Model){
    <tr>
        <td>@item.Name</td>
        <td>@item.Url</td>
        <td>@item.Description</td>
        <td>@item.Label</td>
        <td><a asp-action="Edit" asp-route-id="@item.Id">Upravit</a><a asp-action="Delete" asp-route-id="@item.Id">Smazat</a></td>
    </tr>
    }
    </table>
- ``asp-action="Edit"`` - Zavolá ``IActionResult Edit()``
- ``asp-route-id="@item.Id"`` - Pošle id ``IActionResult Edit(int id)``

#### Load jednoho záznamu (Dapper)
```
using var connection = new SqliteConnection("DataSource=rss.db");
var rss = connection.QueryFirstOrDefault<Rss>("Select * from Rss where Id = @Id", new { Id = id });
return View(rss);
```
#### Formulář ve View
- Třída Rss.cs
```
@model Rss
@using (Html.BeginForm()){
    @Html.LabelFor(x => x.Name)
    @Html.TextBoxFor(x => x.Name)
    @Html.ValidationMessageFor(x => x.Name)
    <button type="submit">Uložit</button>
}
```
```
    @Html.LabelFor(x => x.Label)
    @Html.DropDownListFor(x => x.Label,
        new SelectList(new[]
            {
                "Zprávy",
                "Technologie a věda",
                "Volný čas"
            }))
    @Html.ValidationMessageFor(x => x.Label)
```

#### Controller
- Otevře se při spuštění
```
[HttpGet]
public IActionResult Create(){}
```
- Otevře se po odeslání formuláře
```
[HttpPost]
public IActionResult Create(Rss rss)
```
- Validace formuláře
    ``ModelState.IsValid``
#### Editace
- Musí být ``HidenFor`` aby vědel s čím pracuje (podle Id)
```
@model Rss
@using (Html.BeginForm())
{
    @Html.HiddenFor(x => x.Id)
    <div>
        @Html.LabelFor(x => x.Name)
        @Html.TextBoxFor(x => x.Name)
        @Html.ValidationMessageFor(x => x.Name)
    </div>
    <button type="submit">Uložit změny</button>
}
```
#### XML
```
List<Article> articles = new List<Article>();
foreach(var source in rss)
{
    try {
        var xml = XDocument.Load(source.Url);
        var items = xml.Descendants("item");
        foreach (var item in items) {
            articles.Add(new Article {
                Title = item.Element("title")?.Value,
                Description = item.Element("description")?.Value,
                PubDate = item.Element("pubDate")?.Value
            });
        }
    }catch(Exception) { }
}
```

## XML
- Načtení
```
XDocument doc = XDocument.Load("data.xml"); // .Load(Url)
foreach(var item in doc.Descendants("item")){
    string name = item.Element("title")?.Value;
    int id = int.Parse(item.Element("id")?.Value)
}
```
- Vytvoření
```
XDocument doc = new XDocument(
    new XElement("people", new XElement("person",
                                new XElement("name", "Jan"),
                                new XElement("age", "20"),
                            )
    )
);
doc.Save("output.xml");
```
- Serializace
```
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

Person person = new Person { Name = "Jan", Age = 30 };
XmlSerializer serializer = new XmlSerializer(typeof(Person));
using StreamWriter writer = new StreamWriter("person.xml");
    serializer.serialize(writer, person)
```
- Deserializace
```
using StreamReader reader = new StreamReader("person.xml")
    Person p = (Person)serializer.Deserialize(reader);
    Console.WriteLine(p.Name);
```
## Json
- Třída
```
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```
- Serializace
```
Person person = new Person { Name = "Jan", Age = 30 };
string json = JsonSerializer.Serialize(person);
```
- Deserializace
```
string json = "{\"Name\":\"Jan\",\"Age\":30}";
Person person = JsonSerializer.Deserialize<Person>(json);
```
- **Pole**
```
var people = new Person[]
{
    new Person { Name = "Jan", Age = 30 },
    new Person { Name = "Eva", Age = 25 },
    new Person { Name = "Petr", Age = 40 }
};

//serializace
string json = JsonSerializer.Serialize(people);
//deserializace
Person[] people = JsonSerializer.Deserialize<Person[]>(json);
```
- **List**
```
List<Person> people = JsonSerializer.Deserialize<List<Person>>(json);
```

## HTTPClient
- Get
```
HttpClient client = new HttpClient();
string url = "https://api.example.com/users";
string response = await client.GetStringAsync(url);
```

- Post

```
HttpClient client = new HttpClient();
string json = JsonSerializer.Serialize(person);
StringContent content = new StringContent(json, Encoding.UTF8, "application/json");
HttpResponseMessage response = await client.PostAsync("https://api.example.com/users", content);
string result = await response.Content.ReadAsStringAsync();
```

- Kontrola statusu:
    - ``response.IsSuccessStatusCode``
    - ``response.EnsureSuccesStatusCode()`` - vyhodí vyjímku jestli status není 2xx
- Použití await musí být v async funkci: ``static async Task Main()``
- Hlavičky
```
client.DefaultRequestHeaders.Add("Authorization", "Bearer TOKEN123");
```
- GetStringAsync
    - HttpClient
    - Vrací string
    - Nemá přístup k hlavičkám
    ```
    string content = await client.GetStringAsync("https://jsonplaceholder.typicode.com/posts/1");
    ```
- ReadAsStringAsync
    - HttpContent
    - Vrací string
    - Má přístup k hlavičkám přes HttpResponseMessage
    ```
    HttpResponseMessage response = await client.GetAsync("https://jsonplaceholder.typicode.com/posts/1");
    response.EnsureSuccessStatusCode();
    string content = await response.Content.ReadAsStringAsync();
    ```

## DB
#### Inicializace
- appsettings.json
    - ``"ConnectionStrings": {
    "DefaultConnection": "Data Source=rrs.db"
  }``
- Když chce service tak tam, jinak klidně Main/Program
    - ```
        private string _connectionString = "Data Source=rrs.db";
        private SqliteConnection _connection;
        _connection = new SqliteConnection(_connectionString);
      ```
#### Use Dapper
- ``_connection.Execute("CREATE TABLE A Id INTEGER AUTOINCREMENT;");    //Inicializace``
- ``_connection.Query<Rss>("Select * from Rss").ToList();               //List<Rss>``
- ``_connection.Execute("Insert into Exchange(Name, Email, Currency, Value) VALUES (@Name, @Email, @Currency, @Value)", form);  //Insert``

## WPF
Tag-like struktura
```
<StackPanel>
    <TextBox Name="tbText" MaxLength="100">Text</TextBox>
</StackPanel>
```

``
<Button Name="btnSend" Background="Pink"Click="btnSend_Click">
Odeslat</Button>
``
- Otevření dialogu
    - Pravé tlačítko na řešení -> Add -> Window
    - V kódu
    ```
    MyNewCreatedWindow window = new MyNewCreatedWindow();
    window.Show();
    ```
- Výpis všech záznamů
    - xaml
    ```
    <DataGrid Name="dgCompany" AutoGenerateColumns="True"/>
    ```
    - .cs
    ```
    List<Company> c = ... ;//list, třeba přes Dapper
    dgCompany.ItemSource = c;
    ```
#### Prvky
- ComboBox
```
    <ComboBox Name=cbCity>
        <ComboBoxItem Content="Ostrava"/>
        <ComboBoxItem Content="Praha"/>
        <ComboBoxItem Content="Brno"/>
    </ComboBox>
    //------------ .cs -----------------
    string selected = cbCity.SelectedItem?.ToString();
```
- DataGrid
```
  <DataGrid Name="dgSubjekt" AutoGenerateColumns="False" Height="200">
      <DataGrid.Columns>
          <DataGridTextColumn Header="Název" Binding="{Binding Name}"/>
          <DataGridTextColumn Header="Ičo" Binding="{Binding Ico}"/>
          <DataGridTextColumn Header="Datum" Binding="{Binding Date}"/>
          <DataGridTemplateColumn Header="Editace">
              <DataGridTemplateColumn.CellTemplate>
                  <DataTemplate>
                      <Button Content="Edit" Click="Button_Click_1"/>
                  </DataTemplate>
              </DataGridTemplateColumn.CellTemplate>
          </DataGridTemplateColumn>
      </DataGrid.Columns>
  </DataGrid>
  //------------- .cs -------------------
  dgSubjekt.ItemsSource = subs; //List<Subjekt>;
  //------------ Button click -----------
  private void Button_Click_1(object sender, RoutedEventArgs e)
  {
    var button = sender as Button;
    Subjekt item = button?.DataContext as Subjekt;
  }
```
- "všechny změny se hned projeví" = naschle příští rok :)
```
 public class Subjekt : INotifyPropertyChanged
 {
     private string? _name;
     private int? _ico;
     private string? _date;
     public string? Name
     {
         get => _name;
         set
         {
             _name = value;
             OnPropertyChanged(nameof(Name));
         }
     }
     public int? Ico
     {
         get => _ico;
         set
         {
             _ico = value;
             OnPropertyChanged(nameof(Ico));
         }
     }
     public string? Date
     {
         get => _date;
         set
         {
             _date = value;
             OnPropertyChanged(nameof(Date));
         }
     }
     public event PropertyChangedEventHandler? PropertyChanged;
     protected void OnPropertyChanged(string name)
     {
         PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
     }
 }
 //------------- .xaml -------------------
 <StackPanel>
    <TextBlock>Název</TextBlock>
    <TextBox x:Name="tbName" Text="{Binding Name, UpdateSourceTrigger=PropertyChanged}"/>
    <TextBlock>Ico</TextBlock>
    <TextBox x:Name="tbIco" Text="{Binding Ico, UpdateSourceTrigger=PropertyChanged}"/>
    <TextBlock>Datum</TextBlock>
    <TextBox x:Name="tbDate" Text="{Binding Date, UpdateSourceTrigger=PropertyChanged}"/>
</StackPanel>
```

## Reflexe (je to takzvaně "joever" :skull:)
```
public void Save(Object obj, string path)
{
    if (obj == null) { return; }
    var sb = new StringBuilder();
    var properties = obj.GetType().GetProperties();
    foreach (var prop in properties) {
        if (prop.PropertyType != typeof(string)) continue;
        var value = prop.GetValue(obj) as string;
        sb.AppendLine($"#{prop.Name} => {value};");
    }
    File.WriteAllText(path, sb.ToString());

}
```


## Final random code
```
using System.Text;
using System.Text.Json;
using System.Xml.Linq;
using System.Xml.Serialization;

namespace WebApplication1
{
    public class ReflexionService
    {
        public void Save(Object obj, string path)
        {
            StringBuilder sb = new StringBuilder();
            if (obj == null) { return; }
            var properties = obj.GetType().GetProperties();
            foreach (var property in properties)
            {
                var value = property.GetValue(obj);
                sb.AppendLine($"#{property.Name} => {value};");
            }
            File.WriteAllText(path, sb.ToString());
        }
        public void GetXml()
        {
            //XDocument document = XDocument.Parse("Ahoj"); //From string
            XDocument document = XDocument.Load("Url|Data.xml");
            var children = document.Descendants("Item");
            foreach (var child in children)
            {
                string name = child.Element("name")?.Value;
            }
            XmlSerializer serializer = new XmlSerializer(typeof(string));
            using StreamWriter sw = new StreamWriter("Outputfile.xml");
            serializer.Serialize(sw, document);

            using StreamReader sr = new StreamReader("InputFile.xml");
            string xml = (string)serializer.Deserialize(sr);
        }

        public void GetJson()
        {
            string json = "JsonString";
            List<string> list = JsonSerializer.Deserialize<List<string>>(json);
        }

        public async void Http() {
            HttpClient client = new HttpClient();
            //client.DefaultRequestHeaders.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "token"); //Same thing
            client.DefaultRequestHeaders.Add("Authorization", "Bearer token");
            string json = "String";
            var content = new StringContent(json, Encoding.UTF8, "application/json");
            var response = await client.PostAsync(json, content);
            var content2 = await client.GetStringAsync("Url"); //Pouze obsah
            //I hlavicka a body -> response2.Headers
            var response2 = await client.GetAsync("Url");
            string content3 = await response2.Content.ReadAsStringAsync();
        }
        public void DB() {
        
        }
    }
}

```