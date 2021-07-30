# How to rescue a KiCAD project

Ever updated KiCad or moved a KiCad project to a different location and with 
just a little bit of luck all your symbols, footprints and 3D models are gone?

Yup! Welcome to KiCad reference Hell!

But fear not! Knowing what went wrong and why helps you to recover (most) of 
your work and to avoid these problems in the future.

## What happened?

The KiCad designers, in their infinite wisdom, decided not to embed the parts
you use into your project, but only store references to the global libraries 
they install, using absolute file-path entries.

That of course is a recipe for disaster.

1. If you move your project, you have to adjust all path entries or the 
   references break.
   
2. If someone changes the location of the libraries or their name, the 
   references break (BTW: KiCad and KiCad library updates just love to do that).
   
3. If a part gets changed in a library your design changes too.

4. The whole reference madness makes using git, backups, zip files or other 
   archiving strategies pointless (unless you freeze your whole environment, 
   of course).

5. Since it is impossible to freeze a design, it is also impossible to implement
   a proper review and approve process.

I'm not sure what the guy who came up with this solutions was smoking, but it 
wasn't anything good.

## What we want

We want to:

1. Organize the project in a way, so that we can move it around and share it 
   (e.g. via GitHub), without breaking it, every time the set-up of the new 
   location is not an exact copy of our location.
   
2. Freeze projects in different release states, to be able to review and approve
   revisions using a proper management process and compare different revisions.

## How do we do that?

By doing things the right way and how the program should manage the projects in
the first place.

1. Never, ever store references to global libraries in a project - EVER!

Create a _local_ directory structure for every project and store _all_ 
information for the project here. I use:

```
.\project\
	|
	\3d_models\
	\datasheets\
	\fp-lib.pretty\
	project.kicad_pcb
	project.lib
	project.pro
	project.sch
  
```

and put this under revision control (e.g. and scm like git).

2. Use project local data only

When I draw a new Eeschema, I:

- Create a new library for the _project_ and store it as `project.lib` in the
  project directory;
  
- Copy every schematic symbol I plan to use into this library;

- Use _only_ symbols from the `project.lib` library

When it comes to footprints, I do the same thing:

- Create a new _project_ local library `fp-lib.pretty\`;

- Copy every footprint I plan to use into this library;

- Use _only_ footprints from this library

Data-sheets and 3D models are less important to preserve a design. I follow the
same procedure anyway, just to make sure I find everything, when I need it.

You know you have done everything right, if you can disable all global symbol 
and footprint libraries and your project still loads without problems.

## How to fix an existing project

1. Make a copy and store it in a safe place!

2. REALLY! MAKE A COPY!

3. Open KiCad and disable all global symbol and footprint libraries.

### Rescuing the schematic

4. Load your project into KiCad and try to open the `project.sch` file.
   Most likely all hell breaks loose. 
   
   DON'T PANIC!
   
   There will be warning box about libraries not found. Close them by clicking
   OK.
   
   Finally there will be a `Project Rescue Helper` dialogue. Ignore the 
   description. It's only confusing. Just make sure to press `OK`!
   
What did just happen? Despite my rant in the second chapter there is a local 
copy at least of the symbols used in the schematic. It's called 
`project-cache.lib`. It's just not used as a standard lib. And it is _not_ 
complete. If you compare the just created `project-rescue.lib` with the 
`project-cache.lib` you find that there are parts missing.

Since I'm bothered by having a file named `*-rescue*` in my project. I now 
usually copy all symbols to a new symbol library `project.lib` in the project
directory. For that you can just _copy_ `project-rescue.lib` to `project.lib` 
and add it as a project symbol library to you project.

Now you can either re-assign all symbols in your design to this library or use
your favourite text editor to bulk delete the `-rescue` appendix in all `.sch` 
files of the project.

After that it is save to remove the `project-rescue.lib` from the project and to
delete it.

### Rescuing the footprints

For the footprints there is no automatic backup or rescue function, I know off.
But as long as you still have a good copy of the `project.pcb` file, you can get
your footprints back.

- Create a project local footprint library directory (I usually call it 
  `fp-lib.pretty`).

- Open the `*.pcb` file

- Open the Footprint editor.

- Add the `fp-lib.pretty` to the libraries as a `Project` library.

- Use the `Load Footprints from PCB ...` function in the `Tools` menu:

  - Select one component after the other and save their footprints to your local
    project library (e.g. `fp-lib`)
	
Now you can open the `project.sch` and the footprint assignment editor.

- Delete all footprint associations

- Assign the footprints using the project local library (e.g. `fp-lib`).

### Rescuing Data-sheets and 3D Models

Missing links to Data-sheets and missing 3D Models are more of a cosmetic 
problem. They don't ruin your design.

I usually dump the Data-sheets and models in their respective directories and
re-assign them if needed. Most of the time it is good enough for me to know that 
they are there.
