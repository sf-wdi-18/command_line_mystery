Web Devs Solve Command Line Murders!
====================================
"We didn't know to grep!" - Terminal City PD

Our first task was to go through the crime scene notes and find all the lines marked "CLUE".  These notes were in the crimescene file in the mystery directory.

```
$ cd mystery
$ grep "CLUE" crimescene

CLUE: Footage from an ATM security camera is blurry but shows that the perpetrator is a tall male, at least 6'.
CLUE: Found a wallet believed to belong to the killer: no ID, just loose change, and membership cards for AAA, Delta SkyMiles, the local library, and the Museum of Bash History. The cards are totally untraceable and have no name, for some reason.
CLUE: Questioned the barista at the local coffee shop. He said a woman left right before they heard the shots. The name on her latte was Annabel, she had blond spiky hair and a New Zealand accent.
```

So we have three clues!  Most people's next move was to look for the witness, Annabel, in the people file.

```
$ grep "Annabel" people

Annabel Sun	F	26	Hart Place, line 40
Oluwasegun Annabel	M	37	Mattapan Street, line 173
Annabel Church	F	38	Buckingham Place, line 179
Annabel Fuglsang	M	40	Haley Street, line 176
```

At this point, we don't know which Annabel is our witness, but we can see what happens when we "send police" to someone's address by looking up the indicated line number in the file for their street.  Start by doing that for one person.

``` 
$ head -n 179 streets/Buckingham_Place | tail -n 1
SEE INTERVIEW #699607
```

If you look in the interview directory, you'll see the file names follow the pattern "interview-NUMBER", so let's see what that interview reveals!

```
$ cat interviews/interview-699607

Interviewed Ms. Church at 2:04 pm.  Witness stated that she did not see anyone she could identify as the shooter, that she ran away as soon as the shots were fired.

However, she reports seeing the car that fled the scene.  Describes it as a blue Honda, with a license plate that starts with "L337" and ends with "9"
```

Nice! Our witness!

We could narrow down potential suspects a few ways here, but for now let's use this additional information to filter people based on vehicles they own. 

Take a look in the vehicles file to see how it's formatted:
```
$ head -n 10 vehicles
***************
Vehicle and owner information from the Terminal City Department of Motor Vehicles
***************

License Plate T3YUHF6
Make: Toyota
Color: Yellow
Owner: Jianbo Megannem
Height: 5'6"
Weight: 246 lbs
```

Now we can look through for the manufacturer, license plate, and color we want.  To get the full entry for a car, we'll have to be careful to keep some context before (-B) and after (-A) the lines that match our input string.

```
$ grep "L337" vehicles -A 5

License Plate L337ZR9
Make: Honda
Color: Red
Owner: Katie Park
Height: 6'2"
Weight: 189 lbs
--
License Plate L337P89
Make: Honda
Color: Teal
Owner: Mike Bostock
Height: 6'4"
Weight: 173 lbs
--
License Plate L337GX9
Make: Mazda
Color: Orange
Owner: John Keefe
Height: 6'4"
Weight: 185 lbs
--
.
.
.
.
.
````

We get LOTS of results! Let's narrow it down by looking for the manufacturer we want within those results. Careful to keep context!

```
$ grep "L337" vehicles -A 5 | grep "Honda" -B 1 -A 4 | grep "Blue" -B 2 -A 3

License Plate L337QE9
Make: Honda
Color: Blue
Owner: Erika Owens
Height: 6'5"
Weight: 220 lbs
--
--
License Plate L337539
Make: Honda
Color: Blue
Owner: Aron Pilhofer
Height: 5'3"
Weight: 198 lbs
--
.
.
.
.
.
```

Now remember, our suspect is over 6 feet tall. Add that info.

```
$ grep "L337" vehicles -A 5 | grep "Honda" -B 1 -A 4 | grep "Blue" -B 2 -A 3 | grep "6'" -B 4 -A 1 

License Plate L337QE9
Make: Honda
Color: Blue
Owner: Erika Owens
Height: 6'5"
Weight: 220 lbs
--
--
License Plate L337DV9
Make: Honda
Color: Blue
Owner: Joe Germuska
Height: 6'2"
Weight: 164 lbs
--
--
License Plate L3375A9
Make: Honda
Color: Blue
Owner: Jeremy Bowers
Height: 6'1"
Weight: 204 lbs
--
--
License Plate L337WR9
Make: Honda
Color: Blue
Owner: Jacqui Maher
Height: 6'2"
Weight: 130 lbs
```

By looking in the people file (or guessing based on name), we can see that only a few suspects are male. 

If you recall our clues, we also have membership information. Let's see if one of our suspects is a member of AAA, Delta SkyMiles, the local library, and the Museum of Bash History.

```
$ grep -r "Bowers" membership/

memberships//AAA:Jeremy Bowers
memberships//Delta_SkyMiles:Jeremy Bowers
memberships//Museum_of_Bash_History:Jeremy Bowers
memberships//Terminal_City_Library:Jeremy Bowers
```

We've got him!  