## Step 4 – Executable JAR

### Initial JAR execution

Command executed:

```powershell
java -jar target/fleetcheck-1.0.0.jar
```

### Result

**Execution failed.**

```text
no main manifest attribute, in target\fleetcheck-1.0.0.jar
```

The standard Maven JAR was generated successfully, but it did not contain a `Main-Class` attribute in its manifest.

Therefore, Java could not determine the application's entry point.

### Maven Shade Plugin

The Maven Shade Plugin version `3.6.2` was added to `pom.xml` to create a self-contained executable JAR.

The plugin was configured with:

- Phase: `package`
- Classifier: `all`
- Main class: `pt.upt.fleetcheck.App`

Command executed:

```powershell
mvn clean package
```

**Result: BUILD SUCCESS**

The following executable shaded JAR was generated:

```text
target/fleetcheck-1.0.0-all.jar
```

### First execution of the shaded JAR

Command executed:

```powershell
java -jar target/fleetcheck-1.0.0-all.jar
```

Actual output:

```text
FleetCheck 1.0
Vehicles loaded: 4
Vehicles requiring service: 1
Average mileage: 37000 km
```

Expected output:

```text
FleetCheck 1.0
Vehicles loaded: 4
Vehicles requiring service: 2
Average mileage: 37000 km
```

The shaded JAR executed successfully, but a behavioural defect was identified because only one vehicle was reported as requiring service instead of two.

### Behavioural defect

The defect was located in the `needsService()` method of `FleetService.java`.

Original condition:

```java
return kilometresSinceService > vehicle.serviceIntervalKm();
```

This condition excluded a vehicle when the kilometres since its last service were exactly equal to its service interval.

The condition was corrected to:

```java
return kilometresSinceService >= vehicle.serviceIntervalKm();
```

### Verification after the fix

The project was rebuilt:

```powershell
mvn clean package
```

**Result: BUILD SUCCESS**

The shaded JAR was then executed again:

```powershell
java -jar target/fleetcheck-1.0.0-all.jar
```

Final output:

```text
FleetCheck 1.0
Vehicles loaded: 4
Vehicles requiring service: 2
Average mileage: 37000 km
```

The final output now matches the expected application output.

**Evidence 4: Executable shaded JAR successfully created and behavioural defect corrected.**

## Step 5 – Maven Wrapper

The Maven Wrapper was added to the project so that the build can be executed using the project's configured Maven environment.

The wrapper files were generated, including:

```text
.mvn/wrapper/
mvnw
mvnw.cmd
```

### Wrapper verification

On Windows, the following command was executed:

```powershell
.\mvnw.cmd clean package
```

### Result

**BUILD SUCCESS**

The Maven Wrapper successfully performed the complete build lifecycle, including:

- Cleaning the previous build
- Compiling the application
- Packaging the standard JAR
- Running the Maven Shade Plugin
- Generating the executable shaded JAR

The generated executable artifact is:

```text
target/fleetcheck-1.0.0-all.jar
```

The Maven Shade Plugin produced warnings about overlapping resources, but these warnings did not prevent the build from completing successfully.

This confirms that the project can be built using the Maven Wrapper instead of relying on a globally installed Maven installation.

**Evidence 5: Maven Wrapper build completed successfully.**

## Step 6 – GitHub Actions

Pending.

## Step 7 – SBOM

Pending.