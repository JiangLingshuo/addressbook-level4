# jianglingshuo
###### \java\seedu\address\commons\util\PartialSearchUtil.java
``` java
/** Helper functions to  produce for name examination predicates*/
public class PartialSearchUtil {

    private final ArrayList<String> baseList; // the list of words that is going to be find about
    private final ArrayList<String> targetList; // the list of search words

    //constructor
    public PartialSearchUtil(List<String> baseListOri, List<String> targetListOri) {
        baseList = new ArrayList<>(baseListOri);
        targetList = new ArrayList<>(targetListOri);
    }

    /** Compare two list and see whether any string in baseList contains(partially) any string in targetList*/
    public boolean compare() {
        boolean flag = false;
        ListIterator<String> baseListItr = baseList.listIterator();
        while (baseListItr.hasNext()) {
            String baseString = baseListItr.next();
            ListIterator<String> targetListItr = targetList.listIterator();
            while (targetListItr.hasNext()) {
                if (targetListItr.next().toLowerCase().contains(baseString.toLowerCase())) {
                    flag = true;
                    break;
                }
            }
        }
        return flag;
    }

}
```
###### \java\seedu\address\logic\commands\AddCommand.java
``` java
/**
 * Adds a person to the address book.
 */
public class AddCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "add";
    public static final String COMMAND_ALT = "a";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Adds a person to the address book. "
            + "Parameters: "
            + PREFIX_NAME + "NAME "
            + "[" + PREFIX_PHONE + "PHONE] "
            + "[" + PREFIX_EMAIL + "EMAIL] "
            + "[" + PREFIX_ADDRESS + "ADDRESS] "
            + "[" + PREFIX_TAG + "TAG]...\n"
            + "Example: " + COMMAND_WORD + " "
            + PREFIX_NAME + "John Doe "
            + PREFIX_PHONE + "98765432 "
            + PREFIX_EMAIL + "johnd@example.com "
            + PREFIX_ADDRESS + "311, Clementi Ave 2, #02-25 "
            + PREFIX_TAG + "friends "
            + PREFIX_TAG + "owesMoney ";

    public static final String MESSAGE_SUCCESS = "New person added: %1$s";
    public static final String MESSAGE_DUPLICATE_PERSON = "This person already exists in the address book";

    private final Person toAdd;

```
###### \java\seedu\address\logic\commands\FindCommand.java
``` java
/**
 * Finds and lists all persons in address book whose name contains any of the argument keywords.
 * Keyword matching is case sensitive.
 */
public class FindCommand extends Command {

    public static final String COMMAND_WORD = "find";
    public static final String COMMAND_ALT = "f";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Finds all persons whose names, address, email, "
            + "phone number or tag or contain (or partially) any of "
            + "the specified keywords (case-insensitive) and displays them as a list with index numbers.\n"
            + "Parameters: KEYWORD [MORE_KEYWORDS]...\n"
            + "Example: " + COMMAND_WORD + " alice bob charlie";

    private final Predicate predicate;

    public FindCommand(Predicate predicate) {
        this.predicate = predicate;
    }

    @Override
    public CommandResult execute() {
        model.updateFilteredPersonList(predicate);
        return new CommandResult(getMessageForPersonListShownSummary(model.getFilteredPersonList().size()));
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof FindCommand // instanceof handles nulls
                && this.predicate.equals(((FindCommand) other).predicate)); // state check
    }
}
```
###### \java\seedu\address\logic\parser\AddCommandParser.java
``` java
/**
 * Parses input arguments and creates a new AddCommand object
 */
public class AddCommandParser implements Parser<AddCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the AddCommand
     * and returns an AddCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public AddCommand parse(String args) throws ParseException {
        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_NAME, PREFIX_PHONE, PREFIX_EMAIL, PREFIX_ADDRESS, PREFIX_TAG);

        if (!arePrefixesPresent(argMultimap, PREFIX_NAME)) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, AddCommand.MESSAGE_USAGE));
        }

        try {
            Name name = ParserUtil.parseName(argMultimap.getValue(PREFIX_NAME)).get();
            Phone phone = ParserUtil.parsePhoneAdd(argMultimap.getValue(PREFIX_PHONE)).get();
            Email email = ParserUtil.parseEmailAdd(argMultimap.getValue(PREFIX_EMAIL)).get();
            Address address = ParserUtil.parseAddressAdd(argMultimap.getValue(PREFIX_ADDRESS)).get();
            Set<Tag> tagList = ParserUtil.parseTags(argMultimap.getAllValues(PREFIX_TAG));

            ReadOnlyPerson person = new Person(name, phone, email, address, tagList);

            return new AddCommand(person);
        } catch (IllegalValueException ive) {
            throw new ParseException(ive.getMessage(), ive);
        }
    }

    /**
     * Returns true if none of the prefixes contains empty {@code Optional} values in the given
     * {@code ArgumentMultimap}.
     */
    private static boolean arePrefixesPresent(ArgumentMultimap argumentMultimap, Prefix... prefixes) {
        return Stream.of(prefixes).allMatch(prefix -> argumentMultimap.getValue(prefix).isPresent());
    }

}
```
###### \java\seedu\address\logic\parser\FindCommandParser.java
``` java
/**
 * Parses input arguments and creates a new FindCommand object
 */
public class FindCommandParser implements Parser<FindCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the FindCommand
     * and returns an FindCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public FindCommand parse(String args) throws ParseException {
        args = args.trim();

        String field;
        String trimmedArgs;

        if (args.length() > 2) {
            field = args.substring(0, 2);
            if (!field.substring(1, 2).equals("/")) {
                trimmedArgs = args;
                field = " n/";
            } else {
                field = " " + field;
                trimmedArgs = args.substring(2);
            }
        } else {
            trimmedArgs = args;
            field = " n/";
        }

        if (trimmedArgs.isEmpty()) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, FindCommand.MESSAGE_USAGE));
        }

        String[] keywords = trimmedArgs.split("\\s+");

        FindCommand returnFindCommand = new FindCommand(new NameContainsKeywordsPredicate(Arrays.asList(keywords)));

        switch (field) {
        case " n/":
            returnFindCommand = new FindCommand(new NameContainsKeywordsPredicate(Arrays.asList(keywords)));
            break;
        case " a/":
            returnFindCommand = new FindCommand(new AddressContainsKeywordsPredicate(Arrays.asList(keywords)));
            break;
        case " e/":
            returnFindCommand = new FindCommand(new EmailContainsKeywordsPredicate(Arrays.asList(keywords)));
            break;
        case " t/":
            returnFindCommand = new FindCommand(new TagContainsKeywordsPredicate(Arrays.asList(keywords)));
            break;
        case " p/":
            returnFindCommand = new FindCommand(new PhoneContainsKeywordsPredicate(Arrays.asList(keywords)));
            break;
        default:
            break;
        }

        return returnFindCommand;
    }

}
```
###### \java\seedu\address\logic\parser\ParserUtil.java
``` java
    /**
     * Parses a {@code Optional<String> phone} into an {@code Optional<Phone>} if {@code phone} is present.
     * If {@code Optional<String> phone} does not exist, create an {@code Optional<Phone>} with empty name "---"
     * See header comment of this class regarding the use of {@code Optional} parameters.
     */
    public static Optional<Phone> parsePhoneAdd(Optional<String> phone) throws IllegalValueException {
        return phone.isPresent() ? Optional.of(new Phone(phone.get())) : Optional.of(new Phone(NOT_EXISTING));
    }
```
###### \java\seedu\address\logic\parser\ParserUtil.java
``` java
    /**
     * Parses a {@code Optional<String> address} into an {@code Optional<Address>} if {@code address} is present.
     * If {@code Optional<String> address} does not exist, create an {@code Optional<Address>} with empty name "---"
     * See header comment of this class regarding the use of {@code Optional} parameters.
     */
    public static Optional<Address> parseAddressAdd(Optional<String> address) throws IllegalValueException {
        return address.isPresent() ? Optional.of(new Address(address.get())) : Optional.of(new Address(NOT_EXISTING));
    }
```
###### \java\seedu\address\logic\parser\ParserUtil.java
``` java
    /**
     * Parses a {@code Optional<String> email} into an {@code Optional<Email>} if {@code email} is present.
     * If {@code Optional<String> email} does not exist, create an {@code Optional<Email>} with empty name "---"
     * See header comment of this class regarding the use of {@code Optional} parameters.
     */
    public static Optional<Email> parseEmailAdd(Optional<String> email) throws IllegalValueException {
        return email.isPresent() ? Optional.of(new Email(email.get())) : Optional.of(new Email(NOT_EXISTING));
    }

```
###### \java\seedu\address\logic\parser\ParserUtil.java
``` java
    /**
     * Parses {@code Collection<String> groups} into a {@code Set<Group>}.
     */
    public static Set<Group> parseGroups(Collection<String> groups) throws IllegalValueException {
        requireNonNull(groups);
        final Set<Group> groupSet = new HashSet<>();
        for (String groupName : groups) {
            groupSet.add(new Group(groupName));
        }
        return groupSet;
    }

}
```
###### \java\seedu\address\model\person\AddressContainsKeywordsPredicate.java
``` java
/**
 * Tests that a {@code ReadOnlyPerson}'s {@code Name} matches any of the keywords given.
 */
public class AddressContainsKeywordsPredicate implements Predicate<ReadOnlyPerson> {
    private final List<String> keywords;

    public AddressContainsKeywordsPredicate(List<String> keywords) {
        this.keywords = keywords;
    }

    @Override
    public boolean test(ReadOnlyPerson person) {
        List baseList = Arrays.asList(person.getAddress().value.split(" "));
        PartialSearchUtil mySearch = new PartialSearchUtil(keywords, baseList);
        return mySearch.compare();
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof AddressContainsKeywordsPredicate // instanceof handles nulls
                && this.keywords.equals(((AddressContainsKeywordsPredicate) other).keywords)); // state check
    }

}
```
###### \java\seedu\address\model\person\EmailContainsKeywordsPredicate.java
``` java
/**
 * Tests that a {@code ReadOnlyPerson}'s {@code Name} matches any of the keywords given.
 */
public class EmailContainsKeywordsPredicate implements Predicate<ReadOnlyPerson> {
    private final List<String> keywords;

    public EmailContainsKeywordsPredicate(List<String> keywords) {
        this.keywords = keywords;
    }

    @Override
    public boolean test(ReadOnlyPerson person) {
        List baseList = Arrays.asList(person.getEmail().value.split(" "));
        PartialSearchUtil mySearch = new PartialSearchUtil(keywords, baseList);
        return mySearch.compare();
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof EmailContainsKeywordsPredicate // instanceof handles nulls
                && this.keywords.equals(((EmailContainsKeywordsPredicate) other).keywords)); // state check
    }

}
```
###### \java\seedu\address\model\person\PhoneContainsKeywordsPredicate.java
``` java
/**
 * Tests that a {@code ReadOnlyPerson}'s {@code Name} matches any of the keywords given.
 */
public class PhoneContainsKeywordsPredicate implements Predicate<ReadOnlyPerson> {
    private final List<String> keywords;

    public PhoneContainsKeywordsPredicate(List<String> keywords) {
        this.keywords = keywords;
    }

    @Override
    public boolean test(ReadOnlyPerson person) {
        List baseList = Arrays.asList(person.getPhone().value.split(" "));
        PartialSearchUtil mySearch = new PartialSearchUtil(keywords, baseList);
        return mySearch.compare();
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof PhoneContainsKeywordsPredicate // instanceof handles nulls
                && this.keywords.equals(((PhoneContainsKeywordsPredicate) other).keywords)); // state check
    }

}
```
