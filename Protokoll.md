# WAREHOUSE_DOM_Sharma 26.04.2026
  ## Aufgabenstellung: GK
  
  - Warehouse von letzter Aufgabenstellung verwendet.
  - @Entity Annotation auf @Document verändert und z.B @ManyToOne auf @DocumentReference
  - Jpa Implementations allgemein entfernt, und von JpaRepository/CrudRepository auf MongoRepository umgestiegen.
    
      ### Crud mit Mongo-Shell
    
    **Update**: Um ein Produkt z.B in der Mongo-Shell zu aktualisieren, verwenden wir db.products.updateOne()
    <img width="825" height="139" alt="image" src="https://github.com/user-attachments/assets/401d1ca9-8ba6-4654-9683-817283dd61ef" />
    
  **Delete**: Um z.B mehrere Produkte mit einer Bediengung zu löschen, verwenden wir db.products.deleteMany()
  <img width="462" height="43" alt="image" src="https://github.com/user-attachments/assets/5dd0a3c6-75af-4bd0-8e22-92720efd8fc5" />
 <img width="545" height="30" alt="image" src="https://github.com/user-attachments/assets/73bbd6e5-61aa-4328-a856-aafb0c7a45c0" />
  **Read**: db.products.find()
    <img width="699" height="511" alt="image" src="https://github.com/user-attachments/assets/35676ab8-a25b-401f-9a15-e697ae3c3d35" />
    **Create**: Um ein Datensatz in die Collection hinzuzufügen, verwendet man insertOne()
    <img width="946" height="100" alt="image" src="https://github.com/user-attachments/assets/c25831da-2c1c-4534-8b8f-5df6e97b1b59" />

  ## Aufgabenstellung: EK
  Hier musste ich den Code leicht umschreiben, eine DataGeneratorKlasse erstellen, und @DocumentReference entfernen, weil es sonst zu Problemen kam.
  DataGeneratorEK Code:

public class DataGenerator implements CommandLineRunner {

    private final RestTemplate restTemplate;

    public DataGenerator(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @Override
    public void run(String... args) throws Exception {
        Thread.sleep(3000);

        String baseUrl = "http://localhost:8080";
        List<Warehouse> warehouses = new ArrayList<>();

        String[][] warehouseData = {
                {"001", "Zentrallager Linz", "Linzer Straße 1", "4020", "Linz"},
                {"003", "Hub Wien", "Wiener Hauptstraße 10", "1010", "Wien"},
                {"004", "Logistikzentrum Graz", "Grazer Gasse 5", "8010", "Graz"}
        };

        for (String[] data : warehouseData) {
            Warehouse w = new Warehouse();
            w.setWarehouse_id(data[0]);
            w.setName(data[1]);
            w.setAddress(data[2]);
            w.setPostal_code(data[3]);
            w.setCity(data[4]);
            w.setCountry("Austria");
            w.setTimestamp(java.time.LocalDateTime.now().toString());

            try {
                restTemplate.postForObject(baseUrl + "/warehouse", w, Warehouse.class);
                warehouses.add(w);
                System.out.println("Lager erstellt: " + w.getName());
            } catch (Exception e) {
                System.err.println("Fehler beim Erstellen des Lagers: " + e.getMessage());
            }
        }

        String[] produkte = {"Apfel", "Eier", "Gurken", "Putzfetzen", "Wattepads", "Melonen"};
        String[] kategorien = {"Obst", "TierProdukt", "Gemüse", "Putzen", "Sonstiges", "Obst"};

        for (int i = 0; i < 50; i++) {
            Product p = new Product();
            p.setProduct_id(String.format("P-%04d", i + 1));
            p.setName(produkte[i % produkte.length] + " " + (i + 1));
            p.setProductCategory(kategorien[i % kategorien.length]);
            p.setProductQuantity((int) (Math.random() * 500) + 10);
            p.setProductUnit("Stück");

            Warehouse targetWarehouse = warehouses.get(i % warehouses.size());
            p.setWarehouse(targetWarehouse);
            p.setWarehouseId(targetWarehouse.getWarehouseId());

            try {
                restTemplate.postForObject(baseUrl + "/product", p, Product.class);
                if (i % 10 == 0) System.out.println("Produkt " + (i + 1) + " an " + targetWarehouse.getName() + " gesendet.");
            } catch (Exception e) {
                System.err.println("Fehler beim Senden des Produkts: " + e.getMessage());
            }
        }

        System.out.println("--- Generierung fertig! ---");
    }
}
## Aufgabenstellung Vertiefung
  Hier werden Aggregration-Pipelines in den MongoDB-Queries eingesetzt, also $group, $sort, $match.
  Frage 1: Welcher Lagerstandort hat die wenigsten Produkte der Kategorie Elektronik? (Logistik) 
<img width="691" height="395" alt="image" src="https://github.com/user-attachments/assets/5b974d11-9aed-4c8c-b50c-7d38c4a3ed4b" />
  Frage 2: Welche Warenkategorien haben den höchsten Gesamt-Lagerbestand über alle Standorte hinweg?
  
  <img width="545" height="465" alt="image" src="https://github.com/user-attachments/assets/cad50baf-9bc3-41f1-b3cd-320f25fe6f6b" />
  Frage 3: Von welchen Produkten über alle Standorte hinweg ist weniger als 20 Stück vorhanden?
  
  <img width="550" height="702" alt="image" src="https://github.com/user-attachments/assets/7955784b-1907-42cd-b3a7-99dc9858ce50" />

  ### Bericht und Analyse mit Ollama
  
  - Gradle neu bauen
  - application.properties ändern
  - Service erstellen
  - DataGenerator ändern
    
### DataGenerator.Java
public class DataGenerator implements CommandLineRunner {

    private final RestTemplate restTemplate;
    private final AiReportService aiReportService;

    public DataGenerator(RestTemplate restTemplate, AiReportService aiReportService) {
        this.restTemplate = restTemplate;
        this.aiReportService = aiReportService;
    }

    @Override
    public void run(String... args) throws Exception {
        Thread.sleep(3000);
        String baseUrl = "http://localhost:8080";

        String[][] warehouseData = {
                {"001", "Zentrallager Wien", "Lagerstraße 1", "1010", "Wien"},
                {"002", "Hub Linz", "Industriezeile 10", "4020", "Linz"},
                {"003", "Logistik Graz", "Grazer Gasse 5", "8010", "Graz"},
                {"004", "Nord-Depot Salzburg", "Alpenweg 99", "5020", "Salzburg"},
                {"005", "West-Verteiler Innsbruck", "Bergisel 1", "6020", "Innsbruck"}
        };

        List<Warehouse> warehouseObjects = new ArrayList<>();


        for (String[] data : warehouseData) {
            Warehouse w = new Warehouse();
            w.setWarehouse_id(data[0]);
            w.setName(data[1]);
            w.setAddress(data[2]);
            w.setPostal_code(data[3]);
            w.setCity(data[4]);
            w.setCountry("Austria");
            w.setTimestamp(java.time.LocalDateTime.now().toString());

            try {
                restTemplate.postForObject(baseUrl + "/warehouse", w, Warehouse.class);
                warehouseObjects.add(w);
                System.out.println("Lager erfolgreich gespeichert: " + w.getName());
            } catch (Exception e) {
                System.err.println("Fehler beim Speichern des Lagers: " + e.getMessage());
            }
        }


        String[] katNamen = {"Elektro", "Haushalt", "Lebensmittel", "Garten", "Werkzeug", "Hygiene"};

        String[][] artikelPool = {
                {"Smartphone", "Ladekabel", "Powerbank", "Kopfhörer", "Tablet", "Smartwatch"},
                {"Pfanne", "Besen", "Müllbeutel", "Geschirrset", "Wasserkocher", "Bügeleisen"},
                {"Nudeln", "Reis", "Tomatensauce", "Olivenöl", "Kaffee", "Tee"},
                {"Rasenmäher", "Gartenschlauch", "Spaten", "Blumenerde", "Rechen", "Dünger"},
                {"Hammer", "Schraubenzieher", "Akkubohrer", "Zange", "Säge", "Wasserwaage"},
                {"Seife", "Zahnpasta", "Shampoo", "Duschgel", "Toilettenpapier", "Deo"}
        };

        System.out.println("Generiere 300 Produkte...");
        for (int i = 0; i < 300; i++) {
            int katIndex = i % katNamen.length;
            int artikelIndex = (i / katNamen.length) % artikelPool[katIndex].length;

            Product p = new Product();
            p.setProduct_id(String.format("P-0%4d", i + 1));

            String basisName = artikelPool[katIndex][artikelIndex];
            p.setName(basisName + " Mod. " + (i / 30 + 1));

            p.setProductCategory(katNamen[katIndex]);
            p.setProductQuantity((int) (Math.random() * 200) + 1);
            p.setProductUnit("Stück");


            Warehouse target = warehouseObjects.get(i % warehouseObjects.size());
            p.setWarehouse(target);
            p.setWarehouseId(target.getWarehouseId());

            try {
                restTemplate.postForObject(baseUrl + "/product", p, Product.class);
                if (i % 50 == 0) System.out.println(i + " Produkte übertragen...");
            } catch (Exception e) {
                System.err.println("Fehler beim Senden von Produkt " + (i+1) + ": " + e.getMessage());
            }
        }

        System.out.println("--- DATENGENERIERUNG ERFOLGREICH ABGESCHLOSSEN ---");

        System.out.println("Frage an Llama 3: Welches Lager hat die wenigsten Elektroprodukte?");

        ResponseEntity<List<Product>> response = restTemplate.exchange(
                baseUrl + "/product?category=Elektro",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<Product>>() {}
        );
        List<Product> elektroProdukte = response.getBody();

        if (elektroProdukte != null && !elektroProdukte.isEmpty()) {
            String report = aiReportService.generateReport("Welches Lager hat die wenigsten Elektroprodukte?", elektroProdukte);
            System.out.println("--- BERICHT ---");
            System.out.println(report);
        } else {
            System.out.println("Keine Elektro-Produkte gefunden.");
        }
        System.out.println("Frage an Llama 3: Von welchen Produkten ist weniger als 20 Stück vorhanden?");
        response = restTemplate.exchange(
                baseUrl + "/product?quantityLessThan=20",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<Product>>() {}
        );
        List<Product> knappProdukte = response.getBody();
        String report = aiReportService.generateReport("Von welchen Produkten ist weniger als 20 Stück vorhanden?", knappProdukte);
        System.out.println("--- BERICHT ---");
        System.out.println(report);
        System.out.println("Frage an Llama 3: Welche Warenkategorien haben den höchsten Gesamt-Lagerbestand über alle Standorte hinweg?");
        response = restTemplate.exchange(
                baseUrl + "/product?groupByCategory=true",
                HttpMethod.GET,
                null,
                new ParameterizedTypeReference<List<Product>>() {}
        );
        List<Product> kategorienProdukte = response.getBody();
        report = aiReportService.generateReport("Welche Warenkategorien haben den höchsten Gesamt-Lagerbestand über alle Standorte hinweg?", kategorienProdukte);
        System.out.println("--- BERICHT ---");
        System.out.println(report);

    }
}
### AiReportService.Java
public class AiReportService {

    private static final Logger logger = LoggerFactory.getLogger(AiReportService.class);
    private final OllamaChatClient chatClient;

    public AiReportService(OllamaChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public String generateReport(String question, List<Product> products) {
        String context = products.stream()
                .map(p -> String.format("- %s [%s]: %d %s (Lager: %s)",
                        p.getName(), p.getProductCategory(), p.getProductQuantity(),
                        p.getProductUnit(), p.getWarehouse().getName()))
                .collect(Collectors.joining("\n"));
        
        String prompt = """
                SYSTEM: Du bist ein Logistik-Analyst.
                DATENBASIS:
                %s
                
                AUFGABE: %s
                
                ANTWORT-FORMAT:
                1. Kurze Analyse (Text)
                2. Visualisierung (Nutze Mermaid.js Code für Diagramme, falls hilfreich)
                3. Handlungsempfehlung
                """.formatted(context, question);
        logger.info("--- ANFRAGE AN LLAMA 3 START ---");
        logger.info(prompt);
        logger.info("--- ANFRAGE AN LLAMA 3 ENDE ---");

        return chatClient.call(prompt);
    }
}
### Mermaid Charts
![image.png](attachment:1db1e0bc-b22a-4d0b-9029-1d13f8823b02:image.png)

Die kleine Menge an Reis Mod. 10 [Lebensmittel] (8 Stück) könnte ein Hinweis auf eine Lagerlücke sein. Es wäre ratsam, die Lieferkette oder die Produktionsmenge zu überprüfen, um sicherzustellen, dass genug Reis für die nachfolgenden Perioden verfügbar ist.
Die Menge an Müllbeutel Mod. 9 [Haushalt] (77 Stück) könnte normal sein, aber es wäre ratsam, die Lagerbestände regelmäßig zu überprüfen und den Bedarf anhand von Verkaufszahlen und anderen Faktoren anzupassen.

![image.png](attachment:f00473e6-efd0-4176-9e81-5cb38df063a6:image.png)

Based on the analysis, it appears that Warehouse A (Zentrallager Wien) has the fewest electronic products with 105 items. Therefore, I recommend optimizing inventory management and storage space in this warehouse to ensure efficient use of resources. Additionally, considering the relatively low volume of electronic products in Warehouse A, it may be beneficial to explore opportunities for cost savings or process improvements to further optimize operations.

![image.png](attachment:9785f2f6-4cd2-4260-98b6-0f8a0809f0fc:image.png)

Based on the analysis and visual representation, I recommend that you focus on optimizing inventory management for these three categories. This can include adjusting storage capacity, streamlining logistics, and implementing just-in-time delivery to reduce waste and increase customer satisfaction.

By targeting these high-demand categories, you'll be able to improve your overall inventory turnover rates, reduce costs, and enhance the customer experience


# Fragestellungen

+ Nennen Sie 4 Vorteile eines NoSQL Repository im Gegensatz zu einem relationalen DBMS

  Skalierbarkeit, Verteilbarkeit, Geschwindigkeit, ideal für große Datenmengen.

+ Nennen Sie 4 Nachteile eines NoSQL Repository im Gegensatz zu einem relationalen DBMS

  Komplexere Datenabfragen, fehlende Normalisierung der Daten, mangelnde Datenintegrität (keine Beziehungen) und Komplexität der Datenverwaltung.
  
+ Welche Schwierigkeiten ergeben sich bei der Zusammenführung der Daten?

  Duplikate, inkonsistente Datenstrukutur, fehlende Validierung 
+ Welche Arten von NoSQL Datenbanken gibt es?

  Schlüsselwert, Dokument, Diagramm, breite Spalten.
+ Nennen Sie einen Vertreter für jede Art?

  **MongoDB**: Document
  
  **Amazon DynamoDB**: Key-Value
  
  **Apache Cassandra**: Spalten-orientiert
  
  **ArangoDB**: Diagramm
  
+ Beschreiben Sie die Abkürzungen CA, CP und AP in Bezug auf das CAP Theorem

  Das Cap Theorem besagt, dass eine Datenbank nur zwei von drei Eigenschaften besitzen kann.
  
    **CP**: System kann distributed sein und man kann sich drauf verlassen, dass die aktuellsten Daten geschickt werden.
  
    **CA**: System antwortet mit aktuellste Daten und man kann sich auf hohe Availability verlassen.
  
    **AP**: Verteiltes System mit hohe Availability.
  
    
+ Mit welchem Befehl koennen Sie den Lagerstand eines Produktes aller Lagerstandorte anzeigen.
  
  db.warehouses.find(
  { "productData.productID": "00-443175" },
  { _id:0, "warehouseID":1, "warehouseName":1, "warehouseCity":1, "productData.$":1 }
  )

  
+ Mit welchem Befehl koennen Sie den Lagerstand eines Produktes eines bestimmten Lagerstandortes anzeigen.
  
  db.warehouses.findOne(
  { warehouseID: "1", "productData.productID": "00-443175" },
  { _id:0, warehouseID:1, warehouseName:1, "productData.$":1 }
  )

