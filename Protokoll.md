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

