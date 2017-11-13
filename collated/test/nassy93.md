# nassy93
###### \java\seedu\address\logic\commands\AddFavouriteCommandTest.java
``` java
public class AddFavouriteCommandTest {
    private Model model = new ModelManager(getAltAddressBook(), new UserPrefs());

    @Test
    public void addFavouriteSuccess() throws Exception {
        Person editedPerson = new PersonBuilder().withFavourite(true).withProfPic("Alice Pauline.png")
                .withEmail("alice@example.com").build();
        AddFavouriteCommand addFavouriteCommand = prepareCommand(INDEX_FIRST_PERSON);

        String expectedMessage = String.format(AddFavouriteCommand.MESSAGE_SUCCESS, editedPerson);

        Model expectedModel = new ModelManager(new AddressBook(model.getAddressBook()), new UserPrefs());
        expectedModel.updatePerson(model.getFilteredPersonList().get(0), editedPerson);

        assertCommandSuccess(addFavouriteCommand, model, expectedMessage, expectedModel);
    }
    @Test
    public void invalidPersonIndexFailure() {
        Index outOfBoundIndex = Index.fromOneBased(model.getFilteredPersonList().size() + 1);
        AddFavouriteCommand addFavouriteCommand = prepareCommand(outOfBoundIndex);

        assertCommandFailure(addFavouriteCommand, model, Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
    }
    @Test
    public void duplicatePersonFailure() {
        AddFavouriteCommand addFavouriteCommand = prepareCommand(INDEX_SECOND_PERSON);

        assertCommandFailure(addFavouriteCommand, model, AddFavouriteCommand.MESSAGE_ALREADY_FAVOURITE);
    }
    /**
     * Returns an {@code ResetPictureCommand} with parameters {@code index}
     **/
    private AddFavouriteCommand prepareCommand(Index index) {
        AddFavouriteCommand addFavouriteCommand = new AddFavouriteCommand(index);
        addFavouriteCommand.setData(model, new CommandHistory(), new UndoRedoStack());
        return addFavouriteCommand;
    }
}
```
###### \java\seedu\address\logic\commands\RemoveFavouriteCommandTest.java
``` java
public class RemoveFavouriteCommandTest {
    private Model model = new ModelManager(getAltAddressBook(), new UserPrefs());

    @Test
    public void removeFavouriteSuccess() throws Exception {
        Person editedPerson = new PersonBuilder().withName("Benson Meier")
                .withAddress("311, Clementi Ave 2, #02-25")
                .withEmail("johnd@example.com").withPhone("98765432")
                .withTags("owesMoney", "friends").build();
        RemoveFavouriteCommand removeFavouriteCommand = prepareCommand(INDEX_SECOND_PERSON);

        String expectedMessage = String.format(RemoveFavouriteCommand.MESSAGE_SUCCESS, editedPerson);

        Model expectedModel = new ModelManager(new AddressBook(model.getAddressBook()), new UserPrefs());
        expectedModel.updatePerson(model.getFilteredPersonList().get(1), editedPerson);

        assertCommandSuccess(removeFavouriteCommand, model, expectedMessage, expectedModel);
    }
    @Test
    public void invalidPersonIndexFailure() {
        Index outOfBoundIndex = Index.fromOneBased(model.getFilteredPersonList().size() + 1);
        RemoveFavouriteCommand removeFavouriteCommand = prepareCommand(outOfBoundIndex);

        assertCommandFailure(removeFavouriteCommand, model, Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
    }
    @Test
    public void duplicatePersonFailure() {
        RemoveFavouriteCommand removeFavouriteCommand = prepareCommand(INDEX_FIRST_PERSON);

        assertCommandFailure(removeFavouriteCommand, model, RemoveFavouriteCommand.MESSAGE_ALREADY_NORMAL);
    }
    /**
     * Returns an {@code ResetPictureCommand} with parameters {@code index}
     **/
    private RemoveFavouriteCommand prepareCommand(Index index) {
        RemoveFavouriteCommand removeFavouriteCommand = new RemoveFavouriteCommand(index);
        removeFavouriteCommand.setData(model, new CommandHistory(), new UndoRedoStack());
        return removeFavouriteCommand;
    }
}
```
###### \java\seedu\address\logic\commands\ResetPictureCommandTest.java
``` java
public class ResetPictureCommandTest {

    private Model model = new ModelManager(getAltAddressBook(), new UserPrefs());

    @Test
    public void resetPictureSuccess() throws Exception {
        Person editedPerson = new PersonBuilder().withEmail("alice@example.com").build();
        ResetPictureCommand resetPictureCommand = prepareCommand(INDEX_FIRST_PERSON);

        String expectedMessage = String.format(ResetPictureCommand.MESSAGE_RESETPICTURE_PERSON_SUCCESS, editedPerson);

        Model expectedModel = new ModelManager(new AddressBook(model.getAddressBook()), new UserPrefs());
        expectedModel.updatePerson(model.getFilteredPersonList().get(0), editedPerson);

        assertCommandSuccess(resetPictureCommand, model, expectedMessage, expectedModel);
    }
    @Test
    public void invalidPersonIndexFailure() {
        Index outOfBoundIndex = Index.fromOneBased(model.getFilteredPersonList().size() + 1);
        ResetPictureCommand resetPictureCommand = prepareCommand(outOfBoundIndex);

        assertCommandFailure(resetPictureCommand, model, Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
    }
    @Test
    public void duplicatePersonFailure() {
        ResetPictureCommand resetPictureCommand = prepareCommand(INDEX_SECOND_PERSON);

        assertCommandFailure(resetPictureCommand, model, ResetPictureCommand.MESSAGE_ALREADY_DEFAULT);
    }
    /**
     * Returns an {@code ResetPictureCommand} with parameters {@code index}
     **/
    private ResetPictureCommand prepareCommand(Index index) {
        ResetPictureCommand resetPictureCommand = new ResetPictureCommand(index);
        resetPictureCommand.setData(model, new CommandHistory(), new UndoRedoStack());
        return resetPictureCommand;
    }
}
```
###### \java\seedu\address\logic\parser\AddFavouriteCommandParserTest.java
``` java
public class AddFavouriteCommandParserTest {
    private AddFavouriteCommandParser parser = new AddFavouriteCommandParser();

    @Test
    public void validArgsReturnsAddFavouriteCommand() {
        assertParseSuccess(parser, "1", new AddFavouriteCommand(INDEX_FIRST_PERSON));
    }

    @Test
    public void invalidArgsThrowsParseException() {
        assertParseFailure(parser, "a", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                AddFavouriteCommand.MESSAGE_USAGE));
    }
}
```
###### \java\seedu\address\logic\parser\RemoveFavouriteCommandParserTest.java
``` java
public class RemoveFavouriteCommandParserTest {
    private RemoveFavouriteCommandParser parser = new RemoveFavouriteCommandParser();

    @Test
    public void validArgsReturnsAddFavouriteCommand() {
        assertParseSuccess(parser, "1", new RemoveFavouriteCommand(INDEX_FIRST_PERSON));
    }

    @Test
    public void invalidArgsThrowsParseException() {
        assertParseFailure(parser, "a", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                RemoveFavouriteCommand.MESSAGE_USAGE));
    }
}
```
###### \java\seedu\address\logic\parser\ResetPictureCommandParserTest.java
``` java
public class ResetPictureCommandParserTest {
    private ResetPictureCommandParser parser = new ResetPictureCommandParser();

    @Test
    public void validArgsReturnsAddFavouriteCommand() {
        assertParseSuccess(parser, "1", new ResetPictureCommand(INDEX_FIRST_PERSON));
    }

    @Test
    public void invalidArgsThrowsParseException() {
        assertParseFailure(parser, "a", String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                ResetPictureCommand.MESSAGE_USAGE));
    }
}
```
###### \java\seedu\address\testutil\TypicalPersons.java
``` java
    public static final ReadOnlyPerson ALICEALT = new PersonBuilder().withName("Alice Pauline")
            .withAddress("123, Jurong West Ave 6, #08-111").withEmail("alice@example.com")
            .withPhone("85355255")
            .withProfPic("Alice Pauline.png")
            .withTags("friends").build();
```
###### \java\seedu\address\testutil\TypicalPersons.java
``` java
    public static final ReadOnlyPerson BENSONALT = new PersonBuilder().withName("Benson Meier")
            .withAddress("311, Clementi Ave 2, #02-25")
            .withEmail("johnd@example.com").withPhone("98765432")
            .withFavourite(true)
            .withTags("owesMoney", "friends").build();
```
###### \java\seedu\address\testutil\TypicalPersons.java
``` java
    /**
     * Returns an {@code AddressBook} with all the typical persons.
     */
    public static AddressBook getAltAddressBook() {
        AddressBook ab = new AddressBook();
        for (ReadOnlyPerson person : getAltPersons()) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }
        return ab;
    }

```
###### \java\seedu\address\testutil\TypicalPersons.java
``` java
    public static List<ReadOnlyPerson> getAltPersons() {
        return new ArrayList<>(Arrays.asList(ALICEALT, BENSONALT, CARL, DANIEL, ELLE, FIONA, GEORGE));
    }
```
