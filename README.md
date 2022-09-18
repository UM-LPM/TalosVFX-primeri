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
