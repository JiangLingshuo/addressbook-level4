# cjianhui
###### \java\guitests\guihandles\GroupCardHandle.java
``` java
/**
 * Provides a handle to a person card in the person list panel.
 */
public class GroupCardHandle extends NodeHandle<Node> {
    private static final String ID_FIELD_ID = "#groupId";
    private static final String NAME_FIELD_ID = "#groupName";

    private final Label idLabel;
    private final Label nameLabel;

    public GroupCardHandle(Node cardNode) {
        super(cardNode);
        this.idLabel = getChildNode(ID_FIELD_ID);
        this.nameLabel = getChildNode(NAME_FIELD_ID);
    }

    public String getId() {
        return idLabel.getText();
    }

    public String getName() {
        return nameLabel.getText();
    }

}
```
###### \java\seedu\address\logic\commands\SortCommandTest.java
``` java
/**
 * Contains integration tests (interaction with the Model) and unit tests for {@code DeleteCommand}.
 */
public class SortCommandTest {

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    private Model model = new ModelManager(getTypicalAddressBook(), new UserPrefs());
    private Model emptyModel = new ModelManager(getEmptyAddressBook(), new UserPrefs());
    private Model modelSortedByNameInReverse = new ModelManager(getSortedAddressBook("name", true), new UserPrefs());
    private Model modelSortedByPhone = new ModelManager(getSortedAddressBook("phone", false), new UserPrefs());
    private Model modelSortedByPhoneInReverse = new ModelManager(getSortedAddressBook("phone", true), new UserPrefs());
    private Model modelSortedByEmail = new ModelManager(getSortedAddressBook("email", false), new UserPrefs());
    private Model modelSortedByEmailInReverse = new ModelManager(getSortedAddressBook("email", true), new UserPrefs());
    private Model modelSortedByAddress = new ModelManager(
            getSortedAddressBook("address", false), new UserPrefs());
    private Model modelSortedByAddressInReverse = new ModelManager(
            getSortedAddressBook("address", true), new UserPrefs());

    @Test
    public void constructor_nullSortOrder_throwsNullPointerException() {
        thrown.expect(NullPointerException.class);
        new SortCommand(CliSyntax.PREFIX_NAME.getPrefix(), null);
    }

    @Test
    public void constructor_nullSortField_throwsNullPointerException() {
        thrown.expect(NullPointerException.class);
        new SortCommand(null, true);
    }

    @Test
    public void execute_catchNoPersonsException() throws CommandException {
        thrown.expect(CommandException.class);
        prepareCommand("n/", false, emptyModel).execute();
    }

    @Test
    public void execute_sortByName_success() throws Exception {
        SortCommand sortCommand = prepareCommand("n/", false, model);
        String expectedMessage = String.format(SortCommand.MESSAGE_SORT_PERSON_SUCCESS, "name", "ascending");
        assertCommandSuccess(sortCommand, model, expectedMessage, model);
    }

    @Test
    public void execute_sortByNameInReverseOrder_success() throws Exception {
        SortCommand sortCommand = prepareCommand("n/", true, model);
        String expectedMessage = String.format(SortCommand.MESSAGE_SORT_PERSON_SUCCESS, "name", "descending");
        assertCommandSuccess(sortCommand, model, expectedMessage, modelSortedByNameInReverse);
    }

    @Test
    public void execute_sortByPhone_success() throws Exception {
        SortCommand sortCommand = prepareCommand("p/", false, model);
        String expectedMessage = String.format(SortCommand.MESSAGE_SORT_PERSON_SUCCESS, "phone", "ascending");
        assertCommandSuccess(sortCommand, model, expectedMessage, modelSortedByPhone);
    }

    @Test
    public void execute_sortByPhoneInReverseOrder_success() throws Exception {
        SortCommand sortCommand = prepareCommand("p/", true, model);
        String expectedMessage = String.format(SortCommand.MESSAGE_SORT_PERSON_SUCCESS, "phone", "descending");
        assertCommandSuccess(sortCommand, model, expectedMessage, modelSortedByPhoneInReverse);
    }

    @Test
    public void execute_sortByEmail_success() throws Exception {
        SortCommand sortCommand = prepareCommand("e/", false, model);
        String expectedMessage = String.format(SortCommand.MESSAGE_SORT_PERSON_SUCCESS, "email", "ascending");
        assertCommandSuccess(sortCommand, model, expectedMessage, modelSortedByEmail);
    }

    @Test
    public void execute_sortByEmailInReverseOrder_success() throws Exception {
        SortCommand sortCommand = prepareCommand("e/", true, model);
        String expectedMessage = String.format(SortCommand.MESSAGE_SORT_PERSON_SUCCESS, "email", "descending");
        assertCommandSuccess(sortCommand, model, expectedMessage, modelSortedByEmailInReverse);
    }

    @Test
    public void execute_sortByAddress_success() throws Exception {
        SortCommand sortCommand = prepareCommand("a/", false, model);
        String expectedMessage = String.format(SortCommand.MESSAGE_SORT_PERSON_SUCCESS, "address", "ascending");
        assertCommandSuccess(sortCommand, model, expectedMessage, modelSortedByAddress);
    }

    @Test
    public void execute_sortByAddressInReverseOrder_success() throws Exception {
        SortCommand sortCommand = prepareCommand("a/", true, model);
        String expectedMessage = String.format(SortCommand.MESSAGE_SORT_PERSON_SUCCESS, "address", "descending");
        assertCommandSuccess(sortCommand, model, expectedMessage, modelSortedByAddressInReverse);
    }

    /**
     * Returns a {@code sortCommand} with the parameters {@code field and @code isReverseOrder}.
     */
    private SortCommand prepareCommand(String field, boolean isReverseOrder, Model model) {
        SortCommand sortCommand = new SortCommand(field, isReverseOrder);
        sortCommand.setData(model, new CommandHistory(), new UndoRedoStack());
        return sortCommand;
    }


}
```
###### \java\seedu\address\logic\parser\SortCommandParserTest.java
``` java
public class SortCommandParserTest {

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    private SortCommandParser parser = new SortCommandParser();

    @Test
    public void parse_invalidArgs_failure() {

        //More than 1 field specified
        assertParseFailure(parser, WHITESPACE + CliSyntax.PREFIX_NAME
                         + WHITESPACE + CliSyntax.PREFIX_ADDRESS + SortCommand.REVERSE_ORDER,
                String.format(MESSAGE_INVALID_COMMAND_FORMAT, SortCommand.MESSAGE_USAGE));

        //Invalid field specified
        assertParseFailure(parser, WHITESPACE + "invalid/i",
                String.format(MESSAGE_INVALID_COMMAND_FORMAT, SortCommand.MESSAGE_USAGE));

        //Invalid sort order specified
        assertParseFailure(parser, WHITESPACE + CliSyntax.PREFIX_NAME + "invalid",
                String.format(MESSAGE_INVALID_COMMAND_FORMAT, SortCommand.MESSAGE_USAGE));
    }

    @Test
    public void parse_optionalSortOrderArg_success() {
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_NAME.toString(), new SortCommand("n/", false));
    }

    @Test
    public void parse_validArgs_success() {

        // No arguments supplied
        assertParseSuccess(parser, "", new SortCommand("n/", false));

        // Valid args to sort by name in ascending order
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_NAME.toString(),
                new SortCommand("n/", false));

        // Valid args to sort by name in descending order
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_NAME
                + SortCommand.REVERSE_ORDER, new SortCommand("n/", true));

        // Valid args to sort by phone in ascending order
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_PHONE,
                new SortCommand("p/", false));

        // Valid args to sort by phone in descending order
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_PHONE
                + SortCommand.REVERSE_ORDER, new SortCommand("p/", true));

        // Valid args to sort by email in ascending order
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_EMAIL,
                new SortCommand("e/", false));

        // Valid args to sort by email in descending order
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_EMAIL
                + SortCommand.REVERSE_ORDER, new SortCommand("e/", true));

        // Valid args to sort by address in ascending order
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_ADDRESS,
                new SortCommand("a/", false));

        // Valid args to sort by address in descending order
        assertParseSuccess(parser, WHITESPACE + CliSyntax.PREFIX_ADDRESS
                + SortCommand.REVERSE_ORDER, new SortCommand("a/", true));

    }
}

```
###### \java\seedu\address\testutil\TypicalPersons.java
``` java
    public static AddressBook getSortedAddressBook(String type, boolean isReverseOrder) {
        AddressBook ab = new AddressBook();
        List<ReadOnlyPerson> personList;

        switch(type) {
        case "name":
            personList = getTypicalPersons();
            break;
        case "phone":
            personList = getTypicalPersonsSortedByPhone();
            break;
        case "email":
            personList = getTypicalPersonsSortedByEmail();
            break;
        case "address":
            personList = getTypicalPersonsSortedByAddress();
            break;
        default:
            personList = getTypicalPersons();
        }

        if (isReverseOrder) {
            Collections.reverse(personList);
        }

        for (ReadOnlyPerson person : personList) {
            try {
                ab.addPerson(person);
            } catch (DuplicatePersonException e) {
                assert false : "not possible";
            }
        }

        return ab;
    }

    public static List<ReadOnlyPerson> getTypicalPersonsSortedByPhone() {
        return new ArrayList<>(Arrays.asList(ALICE, DANIEL, ELLE, FIONA, GEORGE, CARL, BENSON));
    }

    public static List<ReadOnlyPerson> getTypicalPersonsSortedByEmail() {
        return new ArrayList<>(Arrays.asList(ALICE, GEORGE, DANIEL, CARL, BENSON, FIONA, ELLE));
    }

    public static List<ReadOnlyPerson> getTypicalPersonsSortedByAddress() {
        return new ArrayList<>(Arrays.asList(DANIEL, ALICE, BENSON, GEORGE, FIONA, ELLE, CARL));
    }


}
```
