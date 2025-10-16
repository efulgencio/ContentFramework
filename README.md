# 📦 ContentFramework — Creación y uso como XCFramework en iOS

Este documento describe cómo crear un **framework iOS con UIKit**, compilarlo en formato **`.xcframework`** y utilizarlo en otro proyecto iOS sin necesidad de arrastrar proyectos en Xcode.

---

## 🚀 Objetivo

1. Crear un framework con UIKit (`ContentFramework`)
2. Compilarlo como **.xcframework** (compatible con dispositivo y simulador)
3. Importarlo en un proyecto iOS (`FrameworkConsumerApp`)
4. Usar una vista pública del framework desde la app

---

## 🧩 1. Crear el Framework UIKit

### Paso 1: Crear el proyecto

1. En **Xcode → File → New → Project…**
2. Selecciona **Framework** (en la sección *Framework & Library*).
3. Nómbralo: `ContentFramework`
4. Interface: *UIKit*, Language: *Swift*.

Estructura generada:

```
ContentFramework/
 └── ContentFramework/
      ├── ContentFramework.h
      ├── Info.plist
      └── (aquí irán tus archivos Swift)
```

---

### Paso 2: Crear una vista pública

Crea un archivo `ContentView.swift` dentro de la carpeta `ContentFramework/ContentFramework/`.

```swift
import UIKit

public class ContentView: UIViewController {

    public override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        
        let label = UILabel()
        label.text = "Contenido Frameworks"
        label.textAlignment = .center
        label.font = UIFont.systemFont(ofSize: 24, weight: .medium)
        label.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(label)
        
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
}
```

> 🔑 Usa `public` en las clases y métodos que quieras exponer fuera del framework.

---

## 🧱 2. Compilar el framework

Generaremos versiones para **iOS Device** y **iOS Simulator** y luego unificaremos ambas en un `.xcframework`.

Abre **Terminal** en la raíz del proyecto y ejecuta:

### Compilar para dispositivo

```bash
xcodebuild archive   -scheme ContentFramework   -destination "generic/platform=iOS"   -archivePath "./build/ios_devices.xcarchive"   SKIP_INSTALL=NO BUILD_LIBRARY_FOR_DISTRIBUTION=YES
```

### Compilar para simulador

```bash
xcodebuild archive   -scheme ContentFramework   -destination "generic/platform=iOS Simulator"   -archivePath "./build/ios_simulator.xcarchive"   SKIP_INSTALL=NO BUILD_LIBRARY_FOR_DISTRIBUTION=YES
```

---

## 🧩 3. Crear el `.xcframework`

Combina ambas versiones en un solo paquete universal:

```bash
xcodebuild -create-xcframework   -framework ./build/ios_devices.xcarchive/Products/Library/Frameworks/ContentFramework.framework   -framework ./build/ios_simulator.xcarchive/Products/Library/Frameworks/ContentFramework.framework   -output ./build/ContentFramework.xcframework
```

Resultado:

```
build/
 └── ContentFramework.xcframework/
```

> ✅ Este es el framework listo para distribución o importación en otros proyectos.

---

## 📱 4. Crear la App que usará el framework

1. Crea un nuevo proyecto iOS UIKit, por ejemplo `FrameworkConsumerApp`.
2. Copia el archivo generado `ContentFramework.xcframework` dentro del proyecto, por ejemplo:
   ```
   FrameworkConsumerApp/Frameworks/ContentFramework.xcframework
   ```

---

## ⚙️ 5. Integrar el framework en Xcode

1. Abre el proyecto `FrameworkConsumerApp.xcodeproj`
2. Selecciona el **target principal**
3. Ve a la pestaña **General**
4. En la sección **Frameworks, Libraries, and Embedded Content**:
   - Pulsa **“+” → Add Other → Add Files…**
   - Selecciona `ContentFramework.xcframework`
   - Marca la opción **“Embed & Sign”**

---

## 🧠 6. Usar el framework en tu código UIKit

En `ViewController.swift`:

```swift
import UIKit
import ContentFramework

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        let contentVC = ContentView()
        contentVC.view.frame = view.bounds
        
        addChild(contentVC)
        view.addSubview(contentVC.view)
        contentVC.didMove(toParent: self)
    }
}
```

Al ejecutar la app, se mostrará en pantalla el texto:
```
Contenido Frameworks
```

---

## ✅ Estructura final

```
FrameworkConsumerApp/
 ├── Frameworks/
 │   └── ContentFramework.xcframework/
 ├── FrameworkConsumerApp.xcodeproj
 └── ViewController.swift
```

---

## 🧰 Opcional: Script para automatizar la compilación

Guarda este script como `build_framework.sh` en la raíz del proyecto `ContentFramework`:

```bash
#!/bin/bash

set -e

SCHEME="ContentFramework"
BUILD_DIR="./build"

xcodebuild archive   -scheme $SCHEME   -destination "generic/platform=iOS"   -archivePath "$BUILD_DIR/ios_devices.xcarchive"   SKIP_INSTALL=NO BUILD_LIBRARY_FOR_DISTRIBUTION=YES

xcodebuild archive   -scheme $SCHEME   -destination "generic/platform=iOS Simulator"   -archivePath "$BUILD_DIR/ios_simulator.xcarchive"   SKIP_INSTALL=NO BUILD_LIBRARY_FOR_DISTRIBUTION=YES

xcodebuild -create-xcframework   -framework "$BUILD_DIR/ios_devices.xcarchive/Products/Library/Frameworks/$SCHEME.framework"   -framework "$BUILD_DIR/ios_simulator.xcarchive/Products/Library/Frameworks/$SCHEME.framework"   -output "$BUILD_DIR/$SCHEME.xcframework"

echo "✅ XCFramework creada en $BUILD_DIR/$SCHEME.xcframework"
```

Ejecuta:
```bash
chmod +x build_framework.sh
./build_framework.sh
```

---

## 📄 Licencia

Uso libre con fines educativos o de demostración.  
© 2025 ContentFramework UIKit Example.
