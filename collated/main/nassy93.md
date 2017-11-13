# nassy93
###### \java\seedu\address\logic\commands\AddFavouriteCommand.java
``` java
/**
 * Marks an indexed person as a favourite in the address book.
 */
public class AddFavouriteCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "fadd";
    public static final String COMMAND_ALT = "fa";
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Marks the person identified by the index number used in the last person listing as a favourite.\n"
            + "Parameters: INDEX (must be a positive integer)\n"
            + "Example: " + COMMAND_WORD;

    public static final String MESSAGE_SUCCESS = "%1$s has been marked as a favourite contact.";
    public static final String MESSAGE_ALREADY_FAVOURITE = "This person is already marked as a favourite.";
    public static final String MESSAGE_DUPLICATE_PERSON = "This person already exists in the address book.";

    private final Index targetIndex;

    public AddFavouriteCommand(Index index) {
        requireNonNull(index);
        this.targetIndex = index;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        List<ReadOnlyPerson> lastShownList = model.getFilteredPersonList();

        if (targetIndex.getZeroBased() >= lastShownList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }

        ReadOnlyPerson personToEdit = lastShownList.get(targetIndex.getZeroBased());

        if (personToEdit.getFavourite().getStatus()) {
            throw new CommandException(MESSAGE_ALREADY_FAVOURITE);
        }

        Person editedPerson = createFavePerson(personToEdit);

        try {
            model.updateFavouritePerson(personToEdit, editedPerson);
        } catch (DuplicatePersonException dpe) {
            throw new CommandException(MESSAGE_DUPLICATE_PERSON);
        } catch (PersonNotFoundException pnfe) {
            throw new AssertionError("The target person cannot be missing");
        }

        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        return new CommandResult(String.format(MESSAGE_SUCCESS, editedPerson));
    }

    /**
     * Creates and returns a {@code Person} with the the Favourite attribute set to true.
     */

    private static Person createFavePerson(ReadOnlyPerson personToEdit) {

        Name updatedName = personToEdit.getName();
        Phone updatedPhone = personToEdit.getPhone();
        Email updatedEmail = personToEdit.getEmail();
        Address updatedAddress = personToEdit.getAddress();
        ProfPic updatedProfPic = personToEdit.getProfPic();
        Favourite updatedFavourite = new Favourite(true);
        Set<Tag> updatedTags = personToEdit.getTags();
        Set<Group> updatedGroups = personToEdit.getGroups();
        Set<Schedule> updatedSchedule = personToEdit.getSchedule();

        return new Person(updatedName, updatedPhone, updatedEmail, updatedAddress, updatedFavourite,
                updatedProfPic, updatedTags, updatedGroups, updatedSchedule);
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof AddFavouriteCommand // instanceof handles nulls
                && this.targetIndex.equals(((AddFavouriteCommand) other).targetIndex)); // state check
    }

}
```
###### \java\seedu\address\logic\commands\RemoveFavouriteCommand.java
``` java
/**
 * Sets Favourite attribute of Indexed person as false in the address book. (remove from favourites)
 */
public class RemoveFavouriteCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "fremove";
    public static final String COMMAND_ALT = "fr";
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Removes the favourite status from the index number used in the last person listing.\n"
            + "Parameters: INDEX (must be a positive integer)\n"
            + "Example: " + COMMAND_WORD + " 1";

    public static final String MESSAGE_SUCCESS = "%1$s has been removed from favourites.";
    public static final String MESSAGE_ALREADY_NORMAL = "This person is not a favourite.";
    public static final String MESSAGE_DUPLICATE_PERSON = "This person already exists in the address book.";

    private final Index targetIndex;

    public RemoveFavouriteCommand(Index index) {
        requireNonNull(index);
        this.targetIndex = index;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        List<ReadOnlyPerson> lastShownList = model.getFilteredPersonList();

        if (targetIndex.getZeroBased() >= lastShownList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }

        ReadOnlyPerson personToEdit = lastShownList.get(targetIndex.getZeroBased());

        if (!personToEdit.getFavourite().getStatus()) {
            throw new CommandException(MESSAGE_ALREADY_NORMAL);
        }

        Person editedPerson = removeFavePerson(personToEdit);

        try {
            model.updateFavouritePerson(personToEdit, editedPerson);
        } catch (DuplicatePersonException dpe) {
            throw new CommandException(MESSAGE_DUPLICATE_PERSON);
        } catch (PersonNotFoundException pnfe) {
            throw new AssertionError("The target person cannot be missing");
        }
        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        return new CommandResult(String.format(MESSAGE_SUCCESS, editedPerson));
    }

    /**
     * Creates and returns a {@code Person} with the the Favourite attribute set to true.
     */
    private static Person removeFavePerson(ReadOnlyPerson personToEdit) {
        Name updatedName = personToEdit.getName();
        Phone updatedPhone = personToEdit.getPhone();
        Email updatedEmail = personToEdit.getEmail();
        Address updatedAddress = personToEdit.getAddress();
        ProfPic updatedProfPic = personToEdit.getProfPic();
        Favourite updatedFavourite = new Favourite(false);
        Set<Tag> updatedTags = personToEdit.getTags();
        Set<Group> updatedGroups = personToEdit.getGroups();
        Set<Schedule> updatedSchedule = personToEdit.getSchedule();

        return new Person(updatedName, updatedPhone, updatedEmail, updatedAddress, updatedFavourite,
                updatedProfPic, updatedTags, updatedGroups, updatedSchedule);
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof RemoveFavouriteCommand // instanceof handles nulls
                && this.targetIndex.equals(((RemoveFavouriteCommand) other).targetIndex)); // state check
    }
}
```
###### \java\seedu\address\logic\commands\ResetPictureCommand.java
``` java
/**
 * Resets ProfPic attribute of Indexed person back to default profile picture in the address book.
 */
public class ResetPictureCommand extends UndoableCommand {
    public static final String COMMAND_WORD = "ppreset";
    public static final String COMMAND_ALT = "ppr";
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Resets the profile picture of the index number used in the last person listing.\n"
            + "Parameters: INDEX (must be a positive integer)\n"
            + "Example: " + COMMAND_WORD + " 1";

    public static final String MESSAGE_RESETPICTURE_PERSON_SUCCESS = "%1$s's profile picture reset to default .";
    public static final String MESSAGE_ALREADY_DEFAULT = "This person's profile picture is already the default.";

    private final Index targetIndex;

    public ResetPictureCommand(Index index) {
        requireNonNull(index);
        this.targetIndex = index;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        List<ReadOnlyPerson> lastShownList = model.getFilteredPersonList();

        if (targetIndex.getZeroBased() >= lastShownList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }

        ReadOnlyPerson personToEdit = lastShownList.get(targetIndex.getZeroBased());
        Person editedPerson;

        try {
            editedPerson = resetProfPicPerson(personToEdit);
            model.updatePerson(personToEdit, editedPerson);
        } catch (DuplicatePersonException dpe) {
            throw new CommandException(MESSAGE_ALREADY_DEFAULT);
        } catch (PersonNotFoundException pnfe) {
            throw new AssertionError("The target person must exist");
        }
        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        return new CommandResult(String.format(MESSAGE_RESETPICTURE_PERSON_SUCCESS, editedPerson));
    }

    /**
     * Creates and returns a {@code Person} with the the ProfPic attribute set to the default picture's path.
     */
    private static Person resetProfPicPerson(ReadOnlyPerson personToEdit) throws DuplicatePersonException {
        if ("maleIcon.png".equals(personToEdit.getProfPic().getPath())) {
            throw new DuplicatePersonException();
        }
        Name updatedName = personToEdit.getName();
        Phone updatedPhone = personToEdit.getPhone();
        Email updatedEmail = personToEdit.getEmail();
        Address updatedAddress = personToEdit.getAddress();
        ProfPic updatedProfPic = new ProfPic("maleIcon.png");
        Favourite updatedFavourite = personToEdit.getFavourite();
        Set<Tag> updatedTags = personToEdit.getTags();
        Set<Group> updatedGroups = personToEdit.getGroups();
        Set<Schedule> updatedSchedule = personToEdit.getSchedule();

        return new Person(updatedName, updatedPhone, updatedEmail, updatedAddress, updatedFavourite,
                updatedProfPic, updatedTags, updatedGroups, updatedSchedule);
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof ResetPictureCommand // instanceof handles nulls
                && this.targetIndex.equals(((ResetPictureCommand) other).targetIndex)); // state check
    }

}
```
###### \java\seedu\address\logic\commands\SetPictureCommand.java
``` java
/**
 * Sets the profile picture for an indexed person.
 */
public class SetPictureCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "ppset";
    public static final String COMMAND_ALT = "pps";
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Changes the profile picture of person in given index with picture at given file path.\n"
            + "Image file must be .png and optimal size 200 x 200.\n"
            + "Parameters: INDEX (must be a positive integer) fp/FILEPATH\n"
            + "Example: " + COMMAND_WORD + " 1 fp/C:\\profilepic.png";

    public static final String MESSAGE_SET_PICTURE_PERSON_SUCCESS = "New profile picture for %1$s has been set";
    public static final String MESSAGE_INVALID_FILE = "File at given file path was not type .png";
    public static final String MESSAGE_FILE_NOT_EXIST = "File does not exist at given file path";

    private static String type;
    private final Index targetIndex;
    private final String filePath;


    public SetPictureCommand(Index index, ProfPic filePath) {
        requireNonNull(index);
        requireNonNull(filePath);
        this.targetIndex = index;
        this.filePath = filePath.getPath();
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        final File file = new File(filePath);
        System.out.println(filePath);
        String fileType;

        List<ReadOnlyPerson> lastShownList = model.getFilteredPersonList();

        if (targetIndex.getZeroBased() >= lastShownList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }
        try {
            fileType = Files.probeContentType(file.toPath());
            if ("image/png".equals(fileType)) { // png verification
                type = ".png";
            } else if ("image/jpeg".equals(fileType)) { // jpg verification
                type = ".jpg";
            } else {
                throw new CommandException(MESSAGE_INVALID_FILE);
            }
        } catch (IOException ioException) {
            throw new CommandException(MESSAGE_FILE_NOT_EXIST);
        }

        ReadOnlyPerson personToEdit = lastShownList.get(targetIndex.getZeroBased());


        // copy picture to resource/image folder and name copied file as PERSON_NAME.png
        Path dest = new File("images/" + personToEdit.getName().toString() + type).toPath();

        try {
            Files.createDirectories(Paths.get("images")); // Creates missing directories if any
            Files.copy(file.toPath(), dest, StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException ioException) {
            // ???
        }

        Person editedPerson = setPicturePerson(personToEdit);

        try {
            model.updatePerson(personToEdit, editedPerson);
        } catch (DuplicatePersonException dpe) { // If duplicate it just means current pic is not default
            return new CommandResult(String.format(MESSAGE_SET_PICTURE_PERSON_SUCCESS, editedPerson));
        } catch (PersonNotFoundException pnfe) {
            throw new AssertionError("The target person cannot be missing");
        }

        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        return new CommandResult(String.format(MESSAGE_SET_PICTURE_PERSON_SUCCESS, editedPerson));
    }

    /**
     * Creates and returns a {@code Person} with the the Favourite attribute set to true.
     */
    private static Person setPicturePerson(ReadOnlyPerson personToEdit) {
        Name updatedName = personToEdit.getName();
        Phone updatedPhone = personToEdit.getPhone();
        Email updatedEmail = personToEdit.getEmail();
        Address updatedAddress = personToEdit.getAddress();
        ProfPic updatedProfPic = new ProfPic(updatedName + type);
        Favourite updatedFavourite = personToEdit.getFavourite();
        Set<Tag> updatedTags = personToEdit.getTags();
        Set<Group> updatedGroups = personToEdit.getGroups();
        Set<Schedule> updatedSchedule = personToEdit.getSchedule();

        return new Person(updatedName, updatedPhone, updatedEmail, updatedAddress, updatedFavourite,
                updatedProfPic, updatedTags, updatedGroups, updatedSchedule);
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof SetPictureCommand // instanceof handles nulls
                && this.targetIndex.equals(((SetPictureCommand) other).targetIndex) // targetIndex state check
                && this.filePath.equals(((SetPictureCommand) other).filePath)); // filePath state check
    }
}
```
###### \java\seedu\address\logic\parser\AddFavouriteCommandParser.java
``` java
/**
 * Parses input arguments and creates a new AddFaveCommand object
 */
public class AddFavouriteCommandParser implements Parser<AddFavouriteCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the AddFaveCommand
     * and returns an AddFaveCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public AddFavouriteCommand parse(String args) throws ParseException {
        try {
            Index index = ParserUtil.parseIndex(args);
            return new AddFavouriteCommand(index);
        } catch (IllegalValueException ive) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, AddFavouriteCommand.MESSAGE_USAGE));
        }
    }
}
```
###### \java\seedu\address\logic\parser\AddressBookParser.java
``` java
        case AddFavouriteCommand.COMMAND_WORD:
        case AddFavouriteCommand.COMMAND_ALT:
            return new AddFavouriteCommandParser().parse(arguments);

        case RemoveFavouriteCommand.COMMAND_WORD:
        case RemoveFavouriteCommand.COMMAND_ALT:
            return new RemoveFavouriteCommandParser().parse(arguments);
```
###### \java\seedu\address\logic\parser\AddressBookParser.java
``` java
        case ResetPictureCommand.COMMAND_WORD:
        case ResetPictureCommand.COMMAND_ALT:
            return new ResetPictureCommandParser().parse(arguments);

        case SetPictureCommand.COMMAND_WORD:
        case SetPictureCommand.COMMAND_ALT:
            return new SetPictureCommandParser().parse(arguments);

        default:
            throw new ParseException(MESSAGE_UNKNOWN_COMMAND);
        }
    }

}
```
###### \java\seedu\address\logic\parser\CliSyntax.java
``` java
    public static final Prefix PREFIX_FAVOURITE = new Prefix("f/");
    public static final Prefix PREFIX_FILEPATH = new Prefix("fp/");
```
###### \java\seedu\address\logic\parser\ParserUtil.java
``` java
    /**
     * Parses a {@code Optional<ProfPic> filePath} into an {@code Optional<ProfPic>} if {@code filePath} is present.
     * See header comment of this class regarding the use of {@code Optional} parameters.
     */
    public static Optional<ProfPic> parseFilePath(Optional<String> filePath) throws IllegalValueException {
        requireNonNull(filePath);
        return filePath.isPresent() ? Optional.of(new ProfPic(filePath.get())) : Optional.empty();
    }

```
###### \java\seedu\address\logic\parser\RemoveFavouriteCommandParser.java
``` java
/**
 * Parses input arguments and creates a new RemoveFaveCommand object
 */
public class RemoveFavouriteCommandParser implements Parser<RemoveFavouriteCommand> {
    /**
     * Parses the given {@code String} of arguments in the context of the RemoveFaveCommand
     * and returns an RemoveFaveCommand object for execution.
     * @throws ParseException if the user input does not conform to the expected format
     */
    public RemoveFavouriteCommand parse(String args) throws ParseException {
        try {
            Index index = ParserUtil.parseIndex(args);
            return new RemoveFavouriteCommand(index);
        } catch (IllegalValueException ive) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, RemoveFavouriteCommand.MESSAGE_USAGE));
        }
    }
}
```
###### \java\seedu\address\logic\parser\ResetPictureCommandParser.java
``` java
/**
 * Parses input arguments and creates a new ResetPictureCommand object
 */
public class ResetPictureCommandParser implements Parser<ResetPictureCommand> {
    /**
     * Parses the given {@code String} of arguments in the context of the ResetPictureCommand
     * and returns an ResetPictureCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public ResetPictureCommand parse(String args) throws ParseException {
        try {
            Index index = ParserUtil.parseIndex(args);
            return new ResetPictureCommand(index);
        } catch (IllegalValueException ive) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, ResetPictureCommand.MESSAGE_USAGE));
        }
    }
}
```
###### \java\seedu\address\logic\parser\SetPictureCommandParser.java
``` java
/**
 * Parses input arguments and creates a new SetPictureCommand object
 */
public class SetPictureCommandParser {
    /**
     * Parses the given {@code String} of arguments in the context of the ResetPictureCommand
     * and returns an ResetPictureCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public SetPictureCommand parse(String args) throws ParseException {
        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_FILEPATH);

        Index index;

        try {
            index = ParserUtil.parseIndex(argMultimap.getPreamble());
        } catch (IllegalValueException ive) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, SetPictureCommand.MESSAGE_USAGE));
        }

        if (!arePrefixesPresent(argMultimap, PREFIX_FILEPATH)) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, SetPictureCommand.MESSAGE_USAGE));
        }

        try {
            ProfPic path = ParserUtil.parseFilePath(argMultimap.getValue(PREFIX_FILEPATH)).get();
            return new SetPictureCommand(index, path);
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
###### \java\seedu\address\model\person\Favourite.java
``` java
/**
 * Stores a person's current "Favourite" status
 *
 */
public class Favourite {
    public final Boolean value;

    public Favourite(Boolean val) {
        value = val;
    }

    public boolean getStatus() {
        return value;
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof Favourite // instanceof handles nulls
                && this.value.equals(((Favourite) other).value)); // state check
    }
}
```
###### \java\seedu\address\model\person\Person.java
``` java
    /**
     * Every field must be present and not null.
     * Constructor for Favourite feature
     */
    public Person(Name name, Phone phone, Email email, Address address, Favourite favourite,
                  ProfPic profPic, Set<Tag> tags) {
        requireAllNonNull(name, phone, email, address, tags);
        this.name = new SimpleObjectProperty<>(name);
        this.phone = new SimpleObjectProperty<>(phone);
        this.email = new SimpleObjectProperty<>(email);
        this.address = new SimpleObjectProperty<>(address);
        this.favourite = new SimpleObjectProperty<>(favourite);
        this.profPic = new SimpleObjectProperty<>(profPic);
        // protect internal tags from changes in the arg list
        this.tags = new SimpleObjectProperty<>(new UniqueTagList(tags));
    }

```
###### \java\seedu\address\model\person\ProfPic.java
``` java
/**
 * Stores the filepath to a person's current profile picture
 *
 */
public class ProfPic {
    public final String path;

    public ProfPic(String val) {
        path = val;
    }

    public String getPath() {
        return path;
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof ProfPic // instanceof handles nulls
                && this.path.equals(((ProfPic) other).path)); // state check
    }
}
```
###### \java\seedu\address\ui\MainContactPanel.java
``` java
    private void setContactImage(ReadOnlyPerson person) throws MalformedURLException {
        Image img;
        if ("maleIcon.png".equals(person.getProfPic().getPath())) {
            img = new Image("images/maleIcon.png");
        } else {
            try {
                File tmp = new File("images/" + person.getProfPic().getPath());
                if (tmp.exists()) {
                    img = new Image(new File("images/" + person.getProfPic().getPath()).toURI().toURL().toString());
                } else { // Failsafe to set contact's image to default if set image is missing
                    img = new Image("images/maleIcon.png");
                }
            } catch (MalformedURLException e) {
                throw new MalformedURLException("URL is malformed in setContactImage()");
            }
        }

        contactImageCircle.setVisible(true);
        contactImageCircle.setFill(new ImagePattern(img));
        easeIn(contactImageCircle);
        currentPerson = person;
    }
```
