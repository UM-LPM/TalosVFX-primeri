Diplomsko delo: https://dk.um.si/IzpisGradiva.php?id=82908&lang=slv

# Primeri učinkov na osnovi delcev izdelanih v Talos VFX

Primeri so izdelani v Talos VFX verziji 1.4.1.
Talos VFX projektne datoteke se nahajajo v mapi "Primeri" uporabljene slike pa v mapi "core/assets".

## Učinek vključimo v igro na naslednji način:

### 1. Vključimo Talos VFX v naš projekt
Dodamo Talos VFX v naš gradle.
```
project(":desktop") {
    apply plugin: "java"
    dependencies {
        compile project(":core")
        compile "com.badlogicgames.gdx:gdx-backend-lwjgl:$gdxVersion"
        compile "com.badlogicgames.gdx:gdx-platform:$gdxVersion:natives-desktop"
        implementation "com.talosvfx:talos-libgdx:1.4.0"  
    }
}
```

Mi smo še imeli problem, da je Talos VFX potreboval jitpack.io tako, da smo na naslednji način še tisto vključili v gradle.

```
repositories {
        mavenLocal()
        mavenCentral()
        jcenter()
        google()
        maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
        maven { url "https://oss.sonatype.org/content/repositories/releases/" }
        maven { url "https://jitpack.io" }
    }
```

### 2. Ustvarimo primeren viewport. 
Temu se lahko izognemo, če upoštevamo velikosti slike med izdelovanjem samega učinka v Talos VFX, vendar je včasih lažje samo kasneje ustvariti primeren viewport za vsak učinek.
```
viewportRocket = new FitViewport(80f, 40f);
viewportRocket.update(Gdx.graphics.getWidth(), Gdx.graphics.getHeight(), false);
```
Vrednosti 80f in 40f je potrebno primerno nastaviti glede na velikost učinka.

### 3. Definiramo batch in renderer
Talos VFX uporablja PolygonSpriteBatch in SpriteBatchParticleRenderer.

```
talosBatch = new PolygonSpriteBatch();
defaultRenderer = new SpriteBatchParticleRenderer();
defaultRenderer.setBatch(talosBatch);
```

### 4. Ustvarimo texture atlas
Pomembno je da se imena regij ujemajo z imeni slik uporabljenih v Talos VFX. Prikazan način ustvarjanja atlasa, ki ga je mogoče uporabiti.
```
TextureRegion textureRegion1 = new TextureRegion(new Texture(Gdx.files.internal("fire.png")));
TextureRegion textureRegion2 = new TextureRegion(new Texture(Gdx.files.internal("rocket64.png")));
TextureRegion textureRegion3 = new TextureRegion(new Texture(Gdx.files.internal("point glow.png")));
TextureRegion textureRegion4 = new TextureRegion(new Texture(Gdx.files.internal("circle.png")));
TextureRegion textureRegion5 = new TextureRegion(new Texture(Gdx.files.internal("astronaut48.png")));
TextureRegion textureRegion6 = new TextureRegion(new Texture(Gdx.files.internal("asteroid128.png")));
TextureRegion textureRegion7 = new TextureRegion(new Texture(Gdx.files.internal("rocket_debris.png")));
TextureRegion textureRegion8 = new TextureRegion(new Texture(Gdx.files.internal("rocket64_front.png")));
TextureRegion textureRegion9 = new TextureRegion(new Texture(Gdx.files.internal("rocket64_left.png")));
TextureRegion textureRegion10 = new TextureRegion(new Texture(Gdx.files.internal("rocket64_right.png")));
TextureRegion textureRegion11 = new TextureRegion(new Texture(Gdx.files.internal("asteroid72.png")));
textureAtlas = new TextureAtlas();
textureAtlas.addRegion("rocket64", textureRegion2);
textureAtlas.addRegion("asteroid128", textureRegion6);
textureAtlas.addRegion("fire", textureRegion1);
textureAtlas.addRegion("point glow", textureRegion3);
textureAtlas.addRegion("circle", textureRegion4);
textureAtlas.addRegion("astronaut48", textureRegion5);
textureAtlas.addRegion("rocket_debris", textureRegion7);
textureAtlas.addRegion("rocket64_front", textureRegion8);
textureAtlas.addRegion("rocket64_left", textureRegion9);
textureAtlas.addRegion("rocket64_right", textureRegion10);
textureAtlas.addRegion("asteroid72", textureRegion11);
```
### 5. Ustvarimo particle effect
Tukaj samo pripravimo učinek in še ga ne upodobimo.
```
EffectDescriptor = new ParticleEffectDescriptor(Gdx.files.internal("rocket.p"), textureAtlas);
ParticleEffectInstance peRocket = EffectDescriptor.createEffectInstance();
```

### 6. Upodobimo učinek
Batch nastavimo, da deluje s primernim viewportom. Nato v Batch-u primerno nastavimo pozicijo učinka, ga posodobimo z funkcijo update() in upodobimo z funkcijo render(). 
```
talosBatch.setProjectionMatrix(viewportRocket.getCamera().projection);
talosBatch.begin();
peRocket.setPosition((float) (rocket.x/12.8 -38.5),-16);
peRocket.update(Gdx.graphics.getDeltaTime());
peRocket.render(defaultRenderer);
talosBatch.end();
```

### Usklajevanje koordinat različnih kamer
Problem nastane pri usklajevanju dveh različno nastavljenih kamer (pri nas med "camera" in drugimi viewporti). Usklajevanje koordinat kamer poteka tako:
Vzamemo velikost prve kamere (pri nas je bila širina 1024) in velikost druge kamere (na primer viewportRocket ima širino 80f) in ju delimo med seboj (1024/80 = 12.8).
To število predstavlja razmerje velikosti med kamerama. 
To lahko nato uporabimo, da pretvorimo koordinate ene kamere v koordinate druge kamere.

```
peRocket.setPosition((float) (rocket.x/12.8 -40 +1.5),-16);
``` 
V našem primeru še je potrebno odšteti polovico velikosti druge kamere (-40) saj so koordinate druge kamere od +40 do -40 (kar je skupaj 80). 
Nato še je včasih potrebno malo ročno nastavljati vrednosti, da se čisto ujemajo (mi smo prišteli 1.5).

## Ustvarjanje učinka v Talos VFX

### 1. Ustvarimo vozlišče Emitter
Z desnim klikom na desni prostor v vmesniku izberemo Emitter vozlišče.
![image](https://user-images.githubusercontent.com/56390707/191081698-36a725b3-5edc-4a71-896d-f60a8d9ea49b.png)

Kliknemo na krogec zraven besede config, da se nam ustvari vozlišče Config, ki ima dodatne lastnosti od Emitterja.
![image](https://user-images.githubusercontent.com/56390707/191081912-230356d2-8ab2-4d42-87f6-7f475a2d1744.png)

### 2. Ustvarimo vozlišče Particle

Na enak način kot za Emitter vozlišče, še ustvarimo vozlišče Particle, ki vsebuje lastnosti delcev, ki jih Emitter proizvaja.

![image](https://user-images.githubusercontent.com/56390707/191082184-02815ac9-d493-4349-9b61-fe8adf42eb15.png)

To je najbolj pomembno vozlišče, ki definira kako izgledajo delci tega Emiterja. Če klikamo na krogece zraven imen lastnosti se nam ustvarijo primerna vozlišča, ki opisujejo lastnost na katero so povezani.

![image](https://user-images.githubusercontent.com/56390707/191082918-2b174679-0900-440e-b067-54e9af5267f7.png)

Bele krogece lahko z miško povlečemo, da spremenimo povezave vozlišč, tako lahko katerikoli vozlišči povežemo med seboj, vendar pravilno delovanje ni zagotovljeno.

![image](https://user-images.githubusercontent.com/56390707/191083549-025f8069-3da3-4f77-9574-f35c36888088.png)

### 3. Ustvarimo več Emitterjev

En Emitter ima lahko samo eno Emitter in Particle vozlišče. Dodatne Emitterje pa ustvarimo tako, da levo spodaj kliknemo na gumb označen na spodnji sliki.

![image](https://user-images.githubusercontent.com/56390707/191084204-219f4461-8d9a-4758-8adc-7cf9fa021d6e.png)

V tem novem Emitterju ponovimo prva dva koraka.

### 4. Izvozimo učinek kot .p datoteko

V menuju File kliknemo na gumb Export As in shranimo .p datoteko, ki jo lahko uporabljamo v libGDX projektih.

![image](https://user-images.githubusercontent.com/56390707/191084412-1980e2b2-4716-46dc-a774-1227859a7c8c.png)

## Dodatne informacije

### Smer potovanja

Lastnost Velocity je direktno povezana z lastnostjo Angle. Če želimo spremeniti smer v katero potujejo delci, moramo spremeniti Angle.

### Vozlišče Color

Z levim klikom ustvarjamo različne točke na barvnem grafu. Z desnim klikom na barvo se nam odpre menu za izbiranje barve.  Delec spreminja barvo čez čas po tem grafu.

![image](https://user-images.githubusercontent.com/56390707/191089650-302df573-b5a7-4b56-a5e3-4c2ffff856f4.png)


### Random Range

Če želimo naključno vrednost je potrebno ustvariti Random Range vozlišče in klikniti označen gumb na sliki.

![image](https://user-images.githubusercontent.com/56390707/191086450-1cd51680-11a6-4a3c-af3b-eb8addbb11b8.png)

Ta izključi povezavo med levo in desno vrednostjo in lahko vnesemo drugačni števili. To vozlišče nato proizvaja naključne vrednosti v nastavljenem razponu.

V Dynamic Random Range vozlišču pa zgornji dve vrednosti definirata razpon naključnih števil vrha desnega grafa, spodnji vrednosti pa razpon naključnih števil podna grafa.

![image](https://user-images.githubusercontent.com/56390707/191087094-828c8a49-8290-47c5-b7a3-4055c8ff5cf9.png)

### Trajanje Emitterja

Če želimo vozlišče povezati s trajanjem Emitterja namesto življenjsko dobo delca potem ustvarimo vozlišče alpha in izberemo Emitter.Alpha - Duration.

![image](https://user-images.githubusercontent.com/56390707/191088966-9afb3a0c-cb3f-4699-9e6c-a744cc17e39c.png)


