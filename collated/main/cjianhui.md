# cjianhui
###### \java\seedu\address\commons\events\ui\GroupPanelSelectionChangedEvent.java
``` java
/**
 * Represents a selection change in the Person List Panel
 */
public class GroupPanelSelectionChangedEvent extends BaseEvent {

    private final GroupCard newSelection;

    public GroupPanelSelectionChangedEvent(GroupCard newSelection) {
        this.newSelection = newSelection;
    }

    @Override
    public String toString() {
        return this.getClass().getSimpleName();
    }

    public GroupCard getNewSelection() {
        return newSelection;
    }
}
```
###### \java\seedu\address\commons\events\ui\JumpToGroupListRequestEvent.java
``` java
/**
 * Indicates a request to jump to the list of persons
 */
public class JumpToGroupListRequestEvent extends BaseEvent {

    public final int targetIndex;

    public JumpToGroupListRequestEvent(Index targetIndex) {
        this.targetIndex = targetIndex.getZeroBased();
    }

    @Override
    public String toString() {
        return this.getClass().getSimpleName();
    }

}
```
###### \java\seedu\address\commons\util\googlecalendarutil\DateParserUtil.java
``` java
/** Helper functions to parse dateTime strings */
public class DateParserUtil {
    private static final String DAY = "Day";
    private static final String MONTH = "Month";
    private static final String DATE = "Date";
    private static final String TIME = "Time";
    private static final String YEAR = "Year";


    /** Convert Google's dateTime string to valid dateTime string */
    public static String convertDateTime(String dateTime) {
        String parsedDateTime = dateTime.replaceFirst("(.*):(..)", "$1$2")
                .replace("T", " ");
        return parsedDateTime;
    }

    /** Parse Google's dateTime and return a hashmap */
    public static HashMap<String, String> parseDateTime(String dateTime) throws ParseException {
        HashMap<String, String> dateTimeMap = new HashMap<>();
        DateFormat dateInput = new SimpleDateFormat("yyyy-MM-dd hh:mm");
        String [] tokens;

        try {
            tokens = dateInput.parse(dateTime).toString().split("\\s+");
        } catch (java.text.ParseException pe) {
            throw new ParseException("Error parsing datetime.");

        }

        dateTimeMap.put(DAY, tokens[0]);
        dateTimeMap.put(MONTH, tokens[1]);
        dateTimeMap.put(DATE, tokens[2]);
        dateTimeMap.put(TIME, tokens[3].substring(0, tokens[3].length() - 3));
        dateTimeMap.put(YEAR, tokens[5]);


        return dateTimeMap;
    }

    /** Ensures event start date is before event end date
     * @param startDateTime
     * @param endDateTime*/
    public static boolean isValidEventDuration(Date startDateTime, Date endDateTime) {
        if (startDateTime.compareTo(endDateTime) > 0) {
            return false;
        }
        return true;
    }

    /** Ensures event start date is after current time
     * @param dateTime*/
    public static boolean isAfterCurrentTime(Date dateTime) {
        Date currentDateTime = new Date();
        if (dateTime.compareTo(currentDateTime) < 0) {
            return false;
        }
        return true;
    }


    /** Sanity checks date input **/
    public static boolean isValidTime(String dateTime) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm");
        sdf.setLenient(false);
        try {
            sdf.parse(dateTime);
        } catch (java.text.ParseException e) {
            return false;
        }

        return true;
    }



    /** Returns yyyy-MM-dd hh:mm representation of current time */
    public static String getCurrentTime() {
        Date currentTime = new Date();
        DateFormat df = new SimpleDateFormat("yyyy-MM-dd hh:mm");
        return df.format(currentTime).toString();
    }



    /** Returns a Google Calendar-like representation of the duration of an event
     * Examples:
     * Events that span more than a day:
     * Saturday, 21 October, 10:30 -
     * Sunday, 22 October, 11:30
     *
     * Events that finish within a day:
     * Sunday, 22 October
     * 08:30 - 12:30
     *
     */
    public static String getDurationOfEvent(String startDateTime, String endDateTime) throws ParseException {
        HashMap<String, String> sDateTimeMap = parseDateTime(startDateTime);
        HashMap<String, String> eDateTimeMap = parseDateTime(endDateTime);
        StringBuilder durationString = new StringBuilder();
        durationString.append(sDateTimeMap.get(DAY) + ", ");
        durationString.append(sDateTimeMap.get(DATE) + " ");
        durationString.append(sDateTimeMap.get(MONTH) + " ");
        durationString.append(sDateTimeMap.get(YEAR));
        if (sDateTimeMap.get(DATE).equals(eDateTimeMap.get(DATE))
                && sDateTimeMap.get(MONTH).equals(eDateTimeMap.get(MONTH))
                && sDateTimeMap.get(YEAR).equals(eDateTimeMap.get(YEAR))) {
            durationString.append("\n" + sDateTimeMap.get(TIME) + " - " + eDateTimeMap.get(TIME));
        } else {
            durationString.append(", " + sDateTimeMap.get(TIME) + " - \n");
            durationString.append(eDateTimeMap.get(DAY) + ", ");
            durationString.append(eDateTimeMap.get(DATE) + " ");
            durationString.append(eDateTimeMap.get(MONTH) + " ");
            durationString.append(eDateTimeMap.get(YEAR) + ", ");
            durationString.append(eDateTimeMap.get(TIME));
        }

        return durationString.toString();
    }

}
```
###### \java\seedu\address\commons\util\googlecalendarutil\EventParserUtil.java
``` java
/** Class to query Google's calendar API to obtain events */
public class EventParserUtil {

    private static final String API = "https://www.googleapis.com/calendar/v3/calendars/";

    /** Node names for Jackson JSON parser to traverse JSON response */
    private static final String EVENTS = "/items";
    private static final String EVENT_NAME = "/summary";
    private static final String EVENT_START = "/start";
    private static final String EVENT_DATE_TIME = "/dateTime";
    private static final String EVENT_END = "/end";
    private static final String EVENT_DETAILS = "/description";
    private static final String KEY = "AIzaSyB34cw8YT02y2qA8ElCddMLxNvS3o1_siI";

    /** Get events ordered by start time */
    private static final String QUERY = "/events?singleEvents=true&orderBy=startTime&key=";

    private static Schedule getSingleSchedule(JsonNode event) throws IllegalValueException {
        String name = event.at(EVENT_NAME).asText();
        String details = event.at(EVENT_DETAILS).asText();
        JsonNode sDateTime = event.at(EVENT_START);
        JsonNode eDateTime = event.at(EVENT_END);
        String startDateTime = DateParserUtil.convertDateTime(sDateTime.at(EVENT_DATE_TIME).asText());
        String endDateTime = DateParserUtil.convertDateTime(eDateTime.at(EVENT_DATE_TIME).asText());
        String duration = DateParserUtil.getDurationOfEvent(startDateTime, endDateTime);
        Schedule schedule = new Schedule(new ScheduleName(name), new ScheduleDate(startDateTime),
                new ScheduleDate(endDateTime), duration, details);
        return schedule;
    }


    public static UniqueScheduleList getScheduleList(String calendarId) throws IOException, IllegalValueException {
        UniqueScheduleList scheduleList = new UniqueScheduleList();
        String apiUrl = API + calendarId + QUERY + KEY;
        String response = ServiceHandlerUtil.makeCall(apiUrl);
        ObjectMapper mapper = new ObjectMapper();
        JsonNode root = mapper.readTree(response);
        JsonNode events = root.at(EVENTS);
        for (JsonNode event: events) {
            scheduleList.add(getSingleSchedule(event));
        }
        return scheduleList;

    }

}
```
###### \java\seedu\address\commons\util\ServiceHandlerUtil.java
``` java
/** Class to make requests to API */
public class ServiceHandlerUtil {

    /**
     * Utility class to make HTTP calls
     * @url - url to make request
     * @method - http request method
     * */
    public static String makeCall(String url) throws IOException {

        URL obj = new URL(url);

        HttpURLConnection connection = (HttpURLConnection) obj.openConnection();
        connection.setRequestMethod("GET");

        return getResponseString(connection);

    }

    public static String getResponseString(HttpURLConnection connection) throws IOException {
        BufferedReader in = new BufferedReader(
                new InputStreamReader(connection.getInputStream()));
        String inputLine;
        StringBuffer response = new StringBuffer();

        while ((inputLine = in.readLine()) != null) {
            response.append(inputLine);
        }
        in.close();
        return response.toString();
    }

}
```
###### \java\seedu\address\logic\commands\AddCalendarCommand.java
``` java
/**
 * Pulls Google Calendar events of a person.
 */
public class AddCalendarCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "cadd";
    public static final String COMMAND_ALT = "ca";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Adds Google Calendar events or a schedule to a person."
            + "Parameters: "
            + "p/PERSON INDEX "
            + PREFIX_CALENDAR_ID + "GOOGLE_CALENDAR_ID" + "\n"
            + "Example: " + COMMAND_WORD + " "
            + "p/2 " + PREFIX_CALENDAR_ID + "xderek105243x@gmail.com";

    public static final String MESSAGE_CALENDAR_PULL_SUCCESS = "Added %1$s Google Calendar Event(s) to %2$s.";
    public static final String MESSAGE_INVALID_CALENDAR_ID = "Google Calendar ID is invalid.";
    public static final String MESSAGE_CALENDAR_PULL_FAIL = "Unable to pull events from Google Calendar.";
    public static final String MESSAGE_DATETIME_ERROR = "Error parsing datetime.";
    public static final String MESSAGE_NO_UPDATE = "%1$s's schedule list is up-to-date.";

    private final Index personIndex;
    private final String calendarId;

    private int addedEvents;
    /**
     * Creates an CreateGroupCommand to add the specified {@code ReadOnlyGroup}
     */
    public AddCalendarCommand(Index personIndex, String calendarId) {
        this.personIndex = personIndex;
        this.calendarId = calendarId;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        requireNonNull(model);

        List<ReadOnlyPerson> lastShownPersonList = model.getFilteredPersonList();

        if (personIndex.getZeroBased() >= lastShownPersonList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }

        ReadOnlyPerson toEdit = lastShownPersonList.get(personIndex.getZeroBased());
        String personName = toEdit.getName().toString();

        Person editedPerson;
        try {
            editedPerson = addCalendarToPerson(toEdit);
        } catch (ParseException e) {
            throw new CommandException(MESSAGE_CALENDAR_PULL_FAIL);
        }

        try {
            model.updatePerson(toEdit, editedPerson);
        } catch (DuplicatePersonException e) {
            throw new AssertionError("The target person cannot exist in address book");
        } catch (PersonNotFoundException e) {
            throw new AssertionError("The target person cannot be missing");
        }
        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);

        if (addedEvents == 0) {
            return new CommandResult(String.format(MESSAGE_NO_UPDATE, personName));
        } else {
            return new CommandResult(String.format(MESSAGE_CALENDAR_PULL_SUCCESS, addedEvents, personName));
        }

    }

    /**
     * Creates and returns a {@code Person} with a schedule list.
     */
    private Person addCalendarToPerson(ReadOnlyPerson personToEdit) throws ParseException, CommandException {
        Name updatedName = personToEdit.getName();
        Phone updatedPhone = personToEdit.getPhone();
        Email updatedEmail = personToEdit.getEmail();
        Address updatedAddress = personToEdit.getAddress();
        Favourite updatedFavourite = personToEdit.getFavourite();
        Set<Tag> updatedTags = personToEdit.getTags();
        UniqueScheduleList updatedScheduleList = personToEdit.scheduleProperty().get();

        try {
            UniqueScheduleList scheduleList = EventParserUtil.getScheduleList(this.calendarId);

            for (Schedule s: scheduleList) {
                try {
                    updatedScheduleList.add(s);
                    addedEvents++;
                } catch (DuplicateScheduleException dse) {
                    continue;
                }
            }

            if (addedEvents > 0) {
                updatedScheduleList.sort();
            }

        } catch (IOException e) {
            throw new CommandException(MESSAGE_INVALID_CALENDAR_ID);
        } catch (IllegalValueException e) {
            throw new CommandException(MESSAGE_DATETIME_ERROR);
        }

        return new Person(updatedName, updatedPhone, updatedEmail, updatedAddress,
                updatedFavourite, updatedTags, updatedScheduleList.toSet());
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof AddCalendarCommand // instanceof handles nulls
                && personIndex.equals(((AddCalendarCommand) other).personIndex));

    }
}
```
###### \java\seedu\address\logic\commands\AddEventCommand.java
``` java
/**
 * Adds a person to the address book.
 */
public class AddEventCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "eadd";
    public static final String COMMAND_ALT = "ea";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Adds an event to a person's schedule. "
            + " \nParameters: "
            + "p/PERSON INDEX "
            + PREFIX_NAME + "EVENT NAME "
            + PREFIX_START_DATE + "EVENT START DATE "
            + PREFIX_END_DATE + "EVENT END DATE "
            + "[" + PREFIX_DETAILS + "EVENT DETAILS]\n"
            + "Date Format: YYYY-MM-DD HH:MM\n"
            + "Example: " + COMMAND_WORD + " p/2 "
            + PREFIX_NAME + "CS2103 Meeting "
            + PREFIX_START_DATE + "2017-11-23 10:30 "
            + PREFIX_END_DATE + "2017-11-23 11:45 "
            + PREFIX_DETAILS + "Prepare for Demo";

    public static final String MESSAGE_SUCCESS = "Added %1$s to %2$s's schedule.";
    public static final String MESSAGE_FAIL = "Unable to add event to %1$s's schedule.";
    public static final String MESSAGE_DUPLICATE_SCHEDULE = "This event already exists in %1$s's schedule.";
    public static final String MESSAGE_INVALID_DURATION = "Please ensure that event "
            + "start time is before event end time.";
    public static final String MESSAGE_INVALID_START_TIME = "Please enter a start time after %1$s.";
    public static final String MESSAGE_INVALID_TIME = "Please enter a valid time.";

    private final Index personIndex;
    private final ReadOnlySchedule schedule;

    /**
     * Creates an AddEventCommand to add the specified {@code ReadOnlySchedule}
     */
    public AddEventCommand(Index personIndex, ReadOnlySchedule toAdd) {
        requireNonNull(personIndex);
        requireNonNull(toAdd);
        this.personIndex = personIndex;
        this.schedule = toAdd;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        requireNonNull(model);

        List<ReadOnlyPerson> lastShownPersonList = model.getFilteredPersonList();

        if (personIndex.getZeroBased() >= lastShownPersonList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }

        ReadOnlyPerson toEdit = lastShownPersonList.get(personIndex.getZeroBased());
        String personName = toEdit.getName().toString();
        String scheduleName = this.schedule.getName().fullName;
        Person editedPerson;
        try {
            editedPerson = addEventToPerson(toEdit, personName);
            model.updatePerson(toEdit, editedPerson);
        } catch (ParseException e) {
            throw new CommandException(String.format(MESSAGE_FAIL, personName));
        } catch (DuplicatePersonException e) {
            throw new AssertionError("The target person cannot exist in address book");
        } catch (PersonNotFoundException e) {
            throw new AssertionError("The target person cannot be missing");
        }
        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        return new CommandResult(String.format(MESSAGE_SUCCESS, scheduleName, personName));

    }

    /**
     * Creates and returns a {@code Person} with a schedule list.
     */
    private Person addEventToPerson(ReadOnlyPerson personToEdit, String personName) throws ParseException {
        Name updatedName = personToEdit.getName();
        Phone updatedPhone = personToEdit.getPhone();
        Email updatedEmail = personToEdit.getEmail();
        Address updatedAddress = personToEdit.getAddress();
        ProfPic updatedProfPic = personToEdit.getProfPic();
        Favourite updatedFavourite = personToEdit.getFavourite();
        Set<Tag> updatedTags = personToEdit.getTags();
        Set<Group> updatedGroups = personToEdit.getGroups();
        UniqueScheduleList updatedScheduleList = personToEdit.scheduleProperty().get();

        try {
            updatedScheduleList.add(this.schedule);
        } catch (DuplicateScheduleException e) {
            throw new ParseException(String.format(MESSAGE_DUPLICATE_SCHEDULE, personName));
        }

        /** Ensure scheduleList is in order **/
        updatedScheduleList.sort();
        return new Person(updatedName, updatedPhone, updatedEmail, updatedAddress, updatedFavourite,
                updatedProfPic, updatedTags, updatedGroups, updatedScheduleList.toSet());
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof AddEventCommand // instanceof handles nulls
                && schedule.equals(((AddEventCommand) other).schedule)
                && personIndex.equals(((AddEventCommand) other).personIndex));

    }
}
```
###### \java\seedu\address\logic\commands\AddPersonToGroupCommand.java
``` java
/**
 * Adds a person to the address book.
 */
public class AddPersonToGroupCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "gadd";
    public static final String COMMAND_ALT = "ga";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Adds a person to a group. "
            + "Parameters: "
            + PREFIX_GROUP + "GROUP INDEX "
            + "p/PERSON INDEX"
            + "Example: " + COMMAND_WORD + " "
            + PREFIX_GROUP + "2" + "p/1";

    public static final String MESSAGE_SUCCESS = "Added %1$s to %2$s.";
    public static final String MESSAGE_DUPLICATE_PERSON = "This person already exists in the group.";

    private final Index personIndex;
    private final Index groupIndex;

    /**
     * Creates an CreateGroupCommand to add the specified {@code ReadOnlyGroup}
     */
    public AddPersonToGroupCommand(Index groupIndex, Index personIndex) {
        this.groupIndex = groupIndex;
        this.personIndex = personIndex;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        requireNonNull(model);

        List<ReadOnlyGroup> lastShownGroupList = model.getFilteredGroupList();
        List<ReadOnlyPerson> lastShownPersonList = model.getFilteredPersonList();

        if (groupIndex.getZeroBased() >= lastShownGroupList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_GROUP_DISPLAYED_INDEX);
        }

        ReadOnlyGroup targetGroup = lastShownGroupList.get(groupIndex.getZeroBased());
        String groupName = targetGroup.getName().toString();

        if (personIndex.getZeroBased() >= lastShownPersonList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }

        ReadOnlyPerson toAdd = lastShownPersonList.get(personIndex.getZeroBased());
        String personName = toAdd.getName().toString();

        try {
            model.addPersonToGroup(groupIndex, toAdd);
            return new CommandResult(String.format(MESSAGE_SUCCESS, personName, groupName));
        } catch (GroupNotFoundException gnfe) {
            assert false : "The target group cannot be missing";
        } catch (PersonNotFoundException pnfe) {
            assert false : "The target person cannot be missing";
        } catch (DuplicatePersonException dpe) {
            throw new CommandException(MESSAGE_DUPLICATE_PERSON);
        }

        return new CommandResult(String.format(MESSAGE_SUCCESS, personName, groupName));

    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof AddPersonToGroupCommand // instanceof handles nulls
                && groupIndex.equals(((AddPersonToGroupCommand) other).groupIndex)
                && personIndex.equals(((AddPersonToGroupCommand) other).personIndex));

    }
}
```
###### \java\seedu\address\logic\commands\CreateGroupCommand.java
``` java
/**
 * Adds a person to the address book.
 */
public class CreateGroupCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "gcreate";
    public static final String COMMAND_ALT = "gc";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Creates a group in address book. "
            + "Parameters: "
            + PREFIX_NAME + "GROUP NAME "
            + "Example: " + COMMAND_WORD + " "
            + PREFIX_NAME + "Bamboo";

    public static final String MESSAGE_SUCCESS = "New group added: %1$s";
    public static final String MESSAGE_DUPLICATE_GROUP = "This group already exists in the address book";

    private final Group toAdd;

    /**
     * Creates an CreateGroupCommand to add the specified {@code ReadOnlyGroup}
     */
    public CreateGroupCommand(ReadOnlyGroup group) {
        toAdd = new Group(group);
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        requireNonNull(model);
        try {
            model.addGroup(toAdd);
            return new CommandResult(String.format(MESSAGE_SUCCESS, toAdd));
        } catch (DuplicateGroupException dge) {
            throw new CommandException(MESSAGE_DUPLICATE_GROUP);
        }

    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof CreateGroupCommand // instanceof handles nulls
                && toAdd.equals(((CreateGroupCommand) other).toAdd));
    }
}
```
###### \java\seedu\address\logic\commands\DeleteEventCommand.java
``` java
/**
 * Removes an event from a person's schedule.
 */
public class DeleteEventCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "edelete";
    public static final String COMMAND_ALT = "ed";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Removes event(s) from a person's schedule. "
            + " \nParameters: "
            + "p/PERSON INDEX "
            + "e/EVENT INDEX [EVENT INDEX...]\n"
            + "Example: " + COMMAND_WORD + " p/2 "
            + "e/1 2 3";

    public static final String MESSAGE_SUCCESS = "Removed %1$s event(s) from %2$s.";
    public static final String MESSAGE_FAIL = "Unable to remove event from %1$s's schedule.";
    public static final String MESSAGE_NO_EVENTS = "%1$s's schedule list is empty.";
    public static final String MESSAGE_NO_SUCH_EVENT = "Event does not exist.";
    private final Index personIndex;
    private Index[] eventIndexes;

    /**
     * Creates an AddEventCommand to add the specified {@code ReadOnlySchedule}
     */
    public DeleteEventCommand(Index personIndex, Index[] eventIndexes) {
        requireNonNull(personIndex);
        requireNonNull(eventIndexes);
        this.personIndex = personIndex;
        this.eventIndexes = eventIndexes;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        requireNonNull(model);

        List<ReadOnlyPerson> lastShownPersonList = model.getFilteredPersonList();

        if (personIndex.getZeroBased() >= lastShownPersonList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }

        ReadOnlyPerson toEdit = lastShownPersonList.get(personIndex.getZeroBased());
        String personName = toEdit.getName().toString();
        int numberOfEvents = this.eventIndexes.length;

        Person editedPerson;
        try {
            editedPerson = removeEventFromPerson(toEdit);
            model.updatePerson(toEdit, editedPerson);
        } catch (ParseException e) {
            throw new CommandException(String.format(MESSAGE_FAIL, personName));
        } catch (DuplicatePersonException e) {
            throw new AssertionError("The target person cannot exist in address book");
        } catch (PersonNotFoundException e) {
            throw new AssertionError("The target person cannot be missing");
        }
        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        return new CommandResult(String.format(MESSAGE_SUCCESS, numberOfEvents, personName));

    }

    /**
     * Creates and returns a {@code Person} with a schedule list.
     */
    private Person removeEventFromPerson(ReadOnlyPerson personToEdit) throws ParseException {
        Name updatedName = personToEdit.getName();
        Phone updatedPhone = personToEdit.getPhone();
        Email updatedEmail = personToEdit.getEmail();
        Address updatedAddress = personToEdit.getAddress();
        Favourite updatedFavourite = personToEdit.getFavourite();
        ProfPic updatedProfPic = personToEdit.getProfPic();
        Set<Tag> updatedTags = personToEdit.getTags();
        Set<Group> updatedGroups = personToEdit.getGroups();
        UniqueScheduleList updatedScheduleList = personToEdit.scheduleProperty().get();

        if (updatedScheduleList.asObservableList().isEmpty()) {
            throw new ParseException(String.format(MESSAGE_NO_EVENTS, updatedName.fullName));
        }

        ReadOnlySchedule[] schedulesToDelete = new ReadOnlySchedule[eventIndexes.length];
        for (int i = 0; i < eventIndexes.length; i++) {
            try {
                schedulesToDelete[i] = updatedScheduleList.asObservableList().get(eventIndexes[i].getZeroBased());
            } catch (IndexOutOfBoundsException e) {
                throw new ParseException(MESSAGE_NO_SUCH_EVENT);
            }
        }

        for (int i = 0; i < eventIndexes.length; i++) {
            try {
                updatedScheduleList.remove(schedulesToDelete[i]);
            } catch (ScheduleNotFoundException e) {
                throw new ParseException(MESSAGE_NO_SUCH_EVENT);
            }
        }

        return new Person(updatedName, updatedPhone, updatedEmail, updatedAddress,
                updatedFavourite, updatedProfPic, updatedTags, updatedGroups,
                updatedScheduleList.toSet());
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof DeleteEventCommand // instanceof handles nulls
                && eventIndexes.equals(((DeleteEventCommand) other).eventIndexes)
                && personIndex.equals(((DeleteEventCommand) other).personIndex));

    }
}
```
###### \java\seedu\address\logic\commands\DeleteGroupCommand.java
``` java
/**
 * Adds a person to the address book.
 */
public class DeleteGroupCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "gdelete";
    public static final String COMMAND_ALT = "gd";

    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Deletes the person identified by the index number used in the last person listing.\n"
            + "Parameters: INDEX (must be a positive integer)\n"
            + "Example: " + COMMAND_WORD + " 1";

    public static final String MESSAGE_DELETE_GROUP_SUCCESS = "Deleted Group: %1$s";

    private final Index targetIndex;

    public DeleteGroupCommand(Index targetIndex) {
        this.targetIndex = targetIndex;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {

        List<ReadOnlyGroup> lastShownList = model.getFilteredGroupList();

        if (targetIndex.getZeroBased() >= lastShownList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_GROUP_DISPLAYED_INDEX);
        }

        ReadOnlyGroup groupToDelete = lastShownList.get(targetIndex.getZeroBased());

        try {
            model.deleteGroup(groupToDelete);
        } catch (GroupNotFoundException gnfe) {
            assert false : "The target group cannot be missing";
        }

        return new CommandResult(String.format(MESSAGE_DELETE_GROUP_SUCCESS, groupToDelete));
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof DeleteGroupCommand // instanceof handles nulls
                && this.targetIndex.equals(((DeleteGroupCommand) other).targetIndex)); // state check
    }
}
```
###### \java\seedu\address\logic\commands\ListCommand.java
``` java
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Lists all persons in the address book.\n"
            + ": Specify prefix f/ to list all person(s) marked as 'Favourite'.\n"
            + "Parameters: [f/]\n"
            + "Example: " + COMMAND_WORD;

    public static final String MESSAGE_SUCCESS = "Listed all persons";
    public static final String MESSAGE_LIST_FAVOURITE_SUCCESS = "Listed all 'favourite' persons";

    private final boolean listFavourite;

    public ListCommand(boolean listFavourite) {
        this.listFavourite = listFavourite;
    }

    @Override
    public CommandResult execute() {
        if (listFavourite) {
            model.updateFilteredPersonList(isFavourite());
            return new CommandResult(MESSAGE_LIST_FAVOURITE_SUCCESS);
        } else {
            model.updateFilteredPersonList(Model.PREDICATE_SHOW_ALL_PERSONS);
            return new CommandResult(MESSAGE_SUCCESS);
        }

    }

    public static Predicate<ReadOnlyPerson> isFavourite() {
        return p ->
                p.getFavourite().getStatus();
    }
}
```
###### \java\seedu\address\logic\commands\RemovePersonFromGroupCommand.java
``` java
/**
 * Adds a person to the address book.
 */
public class RemovePersonFromGroupCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "gremove";
    public static final String COMMAND_ALT = "gr";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Removes a person from a group. "
            + "Parameters: "
            + PREFIX_GROUP + "GROUP INDEX "
            + "p/PERSON INDEX"
            + "Example: " + COMMAND_WORD + " "
            + PREFIX_GROUP + "2" + "p/1";

    public static final String MESSAGE_SUCCESS = "Removed %1$s from %2$s.";
    public static final String MESSAGE_PERSON_NOT_FOUND = "This person does not exist in the group.";
    public static final String MESSAGE_EMPTY_GROUP = "The group is empty.";

    private final Index personIndex;
    private final Index groupIndex;

    /**
     * Creates an CreateGroupCommand to add the specified {@code ReadOnlyGroup}
     */
    public RemovePersonFromGroupCommand(Index groupIndex, Index personIndex) {
        this.groupIndex = groupIndex;
        this.personIndex = personIndex;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {
        requireNonNull(model);

        List<ReadOnlyGroup> lastShownGroupList = model.getFilteredGroupList();
        List<ReadOnlyPerson> lastShownPersonList = model.getFilteredPersonList();

        if (groupIndex.getZeroBased() >= lastShownGroupList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_GROUP_DISPLAYED_INDEX);
        }

        ReadOnlyGroup targetGroup = lastShownGroupList.get(groupIndex.getZeroBased());
        String groupName = targetGroup.getName().toString();

        if (personIndex.getZeroBased() >= lastShownPersonList.size()) {
            throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
        }

        ReadOnlyPerson toAdd = lastShownPersonList.get(personIndex.getZeroBased());
        String personName = toAdd.getName().toString();

        try {
            model.deletePersonFromGroup(groupIndex, toAdd);
            return new CommandResult(String.format(MESSAGE_SUCCESS, personName, groupName));
        } catch (GroupNotFoundException gnfe) {
            assert false : "The target group cannot be missing";
        } catch (PersonNotFoundException pnfe) {
            throw new CommandException(MESSAGE_PERSON_NOT_FOUND);
        } catch (NoPersonsException dpe) {
            throw new CommandException(MESSAGE_EMPTY_GROUP);
        }
        return new CommandResult(String.format(MESSAGE_SUCCESS, personName, groupName));
    }

    @Override
    public boolean equals (Object other) {
        return other == this // short circuit if same object
                || (other instanceof RemovePersonFromGroupCommand // instanceof handles nulls
                && groupIndex.equals(((RemovePersonFromGroupCommand) other).groupIndex)
                && personIndex.equals(((RemovePersonFromGroupCommand) other).personIndex));

    }
}
```
###### \java\seedu\address\logic\commands\SelectCommand.java
``` java
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Selects the person identified by the index number used in the last person listing.\n"
            + ": Specify prefix g/ to select a group by its index number.\n"
            + "Parameters: [g/]INDEX (must be a positive integer)\n"
            + "Example: " + COMMAND_WORD + " 1";

    public static final String MESSAGE_SELECT_PERSON_SUCCESS = "Selected Person: %1$s";
    public static final String MESSAGE_SELECT_GROUP_SUCCESS = "Selected Group: %1$s";

    private final Index targetIndex;
    private final boolean isGroup;

    public SelectCommand(Index targetIndex, boolean isGroup) {
        this.targetIndex = targetIndex;
        this.isGroup = isGroup;
    }

    @Override
    public CommandResult execute() throws CommandException {

        List<ReadOnlyPerson> lastShownList = model.getFilteredPersonList();
        List<ReadOnlyGroup> lastShownGroupList = model.getFilteredGroupList();

        if (isGroup) {
            if (targetIndex.getZeroBased() >= lastShownGroupList.size()) {
                throw new CommandException(Messages.MESSAGE_INVALID_GROUP_DISPLAYED_INDEX);
            }
            EventsCenter.getInstance().post(new JumpToGroupListRequestEvent(targetIndex));
            return new CommandResult(String.format(MESSAGE_SELECT_GROUP_SUCCESS, targetIndex.getOneBased()));
        } else {
            if (targetIndex.getZeroBased() >= lastShownList.size()) {
                throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
            }
            EventsCenter.getInstance().post(new JumpToPersonListRequestEvent(targetIndex));
            return new CommandResult(String.format(MESSAGE_SELECT_PERSON_SUCCESS, targetIndex.getOneBased()));
        }

    }
```
###### \java\seedu\address\logic\commands\SortCommand.java
``` java
/**
 * Sorts persons according to field specified.
 */
public class SortCommand extends UndoableCommand {

    public static final String COMMAND_WORD = "sort";
    public static final String COMMAND_ALT = "s";
    public static final String REVERSE_ORDER = "r";

    public static final String MESSAGE_SORT_PERSON_SUCCESS = "Sorted address book by %1$s in %2$s order.";
    public static final String MESSAGE_EMPTY_LIST = "No person(s) to sort.";

    private static final String PREFIX_NAME_FIELD = "n/";
    private static final String PREFIX_PHONE_FIELD = "p/";
    private static final String PREFIX_EMAIL_FIELD = "e/";
    private static final String PREFIX_ADDRESS_FIELD = "a/";


    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Sorts persons either in ascending or descending order (ascending by default)"
            + " according to prefix specified (name by default)\n"
            + " prefix including n/, a/, e/, p/"
            + "Parameters: "
            + "[PREFIX/[r]]\n"
            + "Example: " + COMMAND_WORD + " "
            + PREFIX_EMAIL_FIELD + REVERSE_ORDER;

    private final String field;
    private final boolean isReverseOrder;

    /*
        Default values assigned to variable used in MESSAGE_SORT_PERSON_SUCCESS
     */
    private String sortBy = "name";
    private String order = "ascending";

    /**
     * @param field     specify which field to sort by
     * @param isReverseOrder specify if sorting is to be in reverse order
     */
    public SortCommand(String field, Boolean isReverseOrder) {
        requireNonNull(field);
        requireNonNull(isReverseOrder);

        this.field = field;
        this.isReverseOrder = isReverseOrder;

    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {

        Comparator<ReadOnlyPerson> sortComparator = getSortComparator(this.field);
        try {
            model.sortPerson(sortComparator, isReverseOrder);
        } catch (NoPersonsException npe) {
            throw new CommandException(MESSAGE_EMPTY_LIST);
        }

        model.updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        if (isReverseOrder) {
            this.order = "descending";
        }
        return new CommandResult(String.format(MESSAGE_SORT_PERSON_SUCCESS, sortBy, order));


    }

    private Comparator<ReadOnlyPerson> getSortComparator(String field) {
        return (o1, o2) -> {
            /** Person(s) marked as 'Favourite' will always remain at the top of the list **/
            if (o1.getFavourite().getStatus()) {
                if (isReverseOrder) {
                    /** Ensure 'Favourite' persons is always greater than when sorted in descending order**/
                    return 1;
                } else {
                    /** Ensure 'Favourite' persons is always smaller than when sorted in ascending order**/
                    return -1;
                }
            }

            switch (field) {
            case PREFIX_NAME_FIELD:
                this.sortBy = "name";
                return o1.getName().toString()
                        .compareToIgnoreCase(o2.getName().toString()
                        );

            case PREFIX_PHONE_FIELD:
                this.sortBy = "phone";
                return o1.getPhone().toString()
                        .compareToIgnoreCase(o2.getPhone().toString()
                        );

            case PREFIX_EMAIL_FIELD:
                this.sortBy = "email";
                return o1.getEmail().toString()
                        .compareToIgnoreCase(o2.getEmail().toString()
                        );

            case PREFIX_ADDRESS_FIELD:
                this.sortBy = "address";
                return o1.getAddress().toString()
                        .compareToIgnoreCase(o2.getAddress().toString()
                        );

            default:
                this.sortBy = "name";
                return o1.getName().toString()
                        .compareToIgnoreCase(o2.getName().toString()
                        );
            }
        };

    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof SortCommand // instanceof handles nulls
                && field.equals(((SortCommand) other).field)
                && REVERSE_ORDER.equals(((SortCommand) other).REVERSE_ORDER));

    }

}


```
###### \java\seedu\address\logic\parser\AddEventCommandParser.java
``` java
/**
 * Parses input arguments and creates a new AddEventCommand object
 */
public class AddEventCommandParser implements Parser<AddEventCommand> {

    public static final Prefix PREFIX_PERSON = new Prefix("p/");

    /**
     * Parses the given {@code String} of arguments in the context of the AddCommand
     * and returns an AddCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public AddEventCommand parse(String args) throws ParseException {
        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_PERSON, PREFIX_NAME,
                        PREFIX_START_DATE, PREFIX_END_DATE, PREFIX_DETAILS);

        String scheduleDetails;

        if (!arePrefixesPresent(argMultimap, PREFIX_PERSON, PREFIX_NAME, PREFIX_START_DATE, PREFIX_END_DATE)) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, AddEventCommand.MESSAGE_USAGE));
        }

        try {
            Index personIndex = ParserUtil.parseIndex(argMultimap.getValue(PREFIX_PERSON).get());
            ScheduleName name = ParserUtil.parseScheduleName(argMultimap.getValue(PREFIX_NAME)).get();
            ScheduleDate sDate = ParserUtil.parseDate(argMultimap.getValue(PREFIX_START_DATE)).get();
            ScheduleDate eDate = ParserUtil.parseDate(argMultimap.getValue(PREFIX_END_DATE)).get();
            scheduleDetails = ParserUtil.parseScheduleDetails(argMultimap.getValue(PREFIX_DETAILS)).get();

            if (!DateParserUtil.isAfterCurrentTime(sDate.scheduleDate)) {
                throw new ParseException(String.format(AddEventCommand.MESSAGE_INVALID_START_TIME,
                        DateParserUtil.getCurrentTime()));
            }

            if (!DateParserUtil.isValidTime(sDate.toString()) || !DateParserUtil.isValidTime(eDate.toString())) {
                throw new ParseException(AddEventCommand.MESSAGE_INVALID_TIME);
            }

            if (!DateParserUtil.isValidEventDuration(sDate.scheduleDate, eDate.scheduleDate)) {
                throw new ParseException(AddEventCommand.MESSAGE_INVALID_DURATION);
            }

            ReadOnlySchedule event = new Schedule(name, sDate, eDate,
                    DateParserUtil.getDurationOfEvent(sDate.toString(), eDate.toString()), scheduleDetails);

            return new AddEventCommand(personIndex, event);
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
###### \java\seedu\address\logic\parser\AddPersonToGroupCommandParser.java
``` java
/**
 * Parses input arguments and creates a new CreateGroupCommand object
 */
public class AddPersonToGroupCommandParser implements Parser<AddPersonToGroupCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the AddCommand
     * and returns an AddCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */

    public static final Prefix PREFIX_PERSON = new Prefix("p/");

    /** Parse AddPersonToGroupCommand Arguments */
    public AddPersonToGroupCommand parse(String args) throws ParseException {
        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_GROUP, PREFIX_PERSON);

        if (!arePrefixesPresent(argMultimap, PREFIX_GROUP, PREFIX_PERSON)) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                    AddPersonToGroupCommand.MESSAGE_USAGE));
        }

        try {
            Index groupIndex = ParserUtil.parseIndex(argMultimap.getValue(PREFIX_GROUP).get());
            Index personIndex = ParserUtil.parseIndex(argMultimap.getValue(PREFIX_PERSON).get());
            return new AddPersonToGroupCommand(groupIndex, personIndex);
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
###### \java\seedu\address\logic\parser\AddressBookParser.java
``` java
        case SortCommand.COMMAND_WORD:
        case SortCommand.COMMAND_ALT:
            return new SortCommandParser().parse(arguments);

        case CreateGroupCommand.COMMAND_WORD:
        case CreateGroupCommand.COMMAND_ALT:
            return new CreateGroupCommandParser().parse(arguments);

        case DeleteGroupCommand.COMMAND_WORD:
        case DeleteGroupCommand.COMMAND_ALT:
            return new DeleteGroupCommandParser().parse(arguments);

        case AddPersonToGroupCommand.COMMAND_WORD:
        case AddPersonToGroupCommand.COMMAND_ALT:
            return new AddPersonToGroupCommandParser().parse(arguments);

        case RemovePersonFromGroupCommand.COMMAND_WORD:
        case RemovePersonFromGroupCommand.COMMAND_ALT:
            return new RemovePersonFromGroupCommandParser().parse(arguments);
```
###### \java\seedu\address\logic\parser\AddressBookParser.java
``` java
        case AddCalendarCommand.COMMAND_WORD:
        case AddCalendarCommand.COMMAND_ALT:
            return new AddScheduleCommandParser().parse(arguments);

        case AddEventCommand.COMMAND_WORD:
        case AddEventCommand.COMMAND_ALT:
            return new AddEventCommandParser().parse(arguments);

        case DeleteEventCommand.COMMAND_WORD:
        case DeleteEventCommand.COMMAND_ALT:
            return new DeleteEventCommandParser().parse(arguments);

```
###### \java\seedu\address\logic\parser\AddScheduleCommandParser.java
``` java
/**
 * Parses input arguments and creates a new AddScheduleCommand object
 */
public class AddScheduleCommandParser implements Parser<AddCalendarCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the AddScheduleCommand
     * and returns an AddScheduleCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */

    public static final Prefix PREFIX_PERSON = new Prefix("p/");

    /** Parse AddSchedueCommand Arguments */
    public AddCalendarCommand parse(String args) throws ParseException {
        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_PERSON, PREFIX_CALENDAR_ID);

        if (!arePrefixesPresent(argMultimap, PREFIX_PERSON, PREFIX_CALENDAR_ID)) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                    AddCalendarCommand.MESSAGE_USAGE));
        }

        try {
            Index personIndex = ParserUtil.parseIndex(argMultimap.getValue(PREFIX_PERSON).get());
            String calendarId = argMultimap.getValue(PREFIX_CALENDAR_ID).get();
            return new AddCalendarCommand(personIndex, calendarId);
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
###### \java\seedu\address\logic\parser\CliSyntax.java
``` java
    public static final Prefix PREFIX_GROUP = new Prefix("g/");
```
###### \java\seedu\address\logic\parser\CliSyntax.java
``` java
    public static final Prefix PREFIX_CALENDAR_ID = new Prefix("i/");
    public static final Prefix PREFIX_START_DATE = new Prefix("s/");
    public static final Prefix PREFIX_END_DATE = new Prefix("e/");
    public static final Prefix PREFIX_DETAILS = new Prefix("d/");
```
###### \java\seedu\address\logic\parser\CreateGroupCommandParser.java
``` java
/**
 * Parses input arguments and creates a new CreateGroupCommand object
 */
public class CreateGroupCommandParser implements Parser<CreateGroupCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the AddCommand
     * and returns an AddCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public CreateGroupCommand parse(String args) throws ParseException {
        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_NAME);

        if (!arePrefixesPresent(argMultimap, PREFIX_NAME)) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, CreateGroupCommand.MESSAGE_USAGE));
        }

        try {
            GroupName name = ParserUtil.parseGroupName(argMultimap.getValue(PREFIX_NAME)).get();
            ReadOnlyGroup group = new Group(name);
            return new CreateGroupCommand(group);
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
###### \java\seedu\address\logic\parser\DeleteEventCommandParser.java
``` java
/**
 * Parses input arguments and creates a new DeleteEventCommand object
 */
public class DeleteEventCommandParser implements Parser<DeleteEventCommand> {



    public static final Prefix PREFIX_PERSON = new Prefix("p/");
    public static final Prefix PREFIX_EVENT = new Prefix("e/");

    /**
     * Parses the given {@code String} of arguments in the context of the DeleteEventCommand
     * and returns an DeleteEventCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public DeleteEventCommand parse(String args) throws ParseException {

        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_PERSON, PREFIX_EVENT);

        if (!arePrefixesPresent(argMultimap, PREFIX_PERSON, PREFIX_EVENT)) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                    DeleteEventCommand.MESSAGE_USAGE));
        }

        Index[] eventIndexes;

        try {
            Index personIndex = ParserUtil.parseIndex(argMultimap.getValue(PREFIX_PERSON).get());
            String[] arguments = argMultimap.getValue(PREFIX_EVENT).get().trim().split(" ");
            eventIndexes = new Index[arguments.length];
            for (int i = 0; i < eventIndexes.length; i++) {
                eventIndexes[i] = ParserUtil.parseIndex(arguments[i]);
            }
            return new DeleteEventCommand(personIndex, eventIndexes);
        } catch (IllegalValueException ive) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, DeleteEventCommand.MESSAGE_USAGE));
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
###### \java\seedu\address\logic\parser\DeleteGroupCommandParser.java
``` java
/**
 * Parses input arguments and creates a new DeleteGroundCommand object
 */
public class DeleteGroupCommandParser implements Parser<DeleteGroupCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the DeleteCommand
     * and returns an DeleteCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */
    public DeleteGroupCommand parse(String args) throws ParseException {
        try {
            Index index = ParserUtil.parseIndex(args);
            return new DeleteGroupCommand(index);
        } catch (IllegalValueException ive) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, DeleteGroupCommand.MESSAGE_USAGE));
        }
    }

}
```
###### \java\seedu\address\logic\parser\ListCommandParser.java
``` java
    private Boolean isFavourite = false;

    /**
     * Parses the given {@code String} of arguments in the context of the SelectCommand
     * and returns an SelectCommand object for execution.
     *
     * @throws ParseException if the user input does not conform the expected format
     */
    public ListCommand parse(String args) throws ParseException {
        if (!args.matches("^( f/)?$")) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, ListCommand.MESSAGE_USAGE));
        }

        if (args.trim().contains(PREFIX_FAVOURITE.getPrefix())) {
            isFavourite = true;
        }
        return new ListCommand(isFavourite);


    }
}
```
###### \java\seedu\address\logic\parser\ParserUtil.java
``` java
    /**
     * Parses a {@code Optional<String> groupName} into an {@code Optional<GroupName>} if {@code groupName} is present.
     * See header comment of this class regarding the use of {@code Optional} parameters.
     */
    public static Optional<GroupName> parseGroupName(Optional<String> groupName) throws IllegalValueException {
        requireNonNull(groupName);
        return groupName.isPresent() ? Optional.of(new GroupName(groupName.get())) : Optional.empty();
    }

    /**
     * Parses a {@code Optional<String> scheduleName} into an {@code Optional<GroupName>}
     * if {@code scheduleName} is present.
     * See header comment of this class regarding the use of {@code Optional} parameters.
     */
    public static Optional<ScheduleName> parseScheduleName(Optional<String> scheduleName) throws IllegalValueException {
        requireNonNull(scheduleName);
        return scheduleName.isPresent() ? Optional.of(new ScheduleName((scheduleName.get()))) : Optional.empty();
    }

    /**
     * Parses a {@code Optional<String> scheduleDate} into an {@code Optional<GroupName>}
     * if {@code scheduleDate} is present.
     * See header comment of this class regarding the use of {@code Optional} parameters.
     */
    public static Optional<ScheduleDate> parseDate(Optional<String> scheduleDate) throws IllegalValueException {
        requireNonNull(scheduleDate);
        return scheduleDate.isPresent() ? Optional.of(new ScheduleDate((scheduleDate.get()))) : Optional.empty();
    }

    /**
     * Parses a {@code Optional<String> scheduleDetails} into an {@code Optional<GroupName>}
     * if {@code scheduleDate} is present.
     * See header comment of this class regarding the use of {@code Optional} parameters.
     */
    public static Optional<String> parseScheduleDetails(Optional<String> scheduleDetails) throws IllegalValueException {
        requireNonNull(scheduleDetails);
        return scheduleDetails.isPresent() ? Optional.of((scheduleDetails.get())) : Optional.empty();
    }
```
###### \java\seedu\address\logic\parser\RemovePersonFromGroupCommandParser.java
``` java
/**
 * Parses input arguments and creates a new CreateGroupCommand object
 */
public class RemovePersonFromGroupCommandParser implements Parser<RemovePersonFromGroupCommand> {

    /**
     * Parses the given {@code String} of arguments in the context of the AddCommand
     * and returns an AddCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */

    public static final Prefix PREFIX_PERSON = new Prefix("p/");

    /** Parse AddPersonToGroupCommand Arguments */
    public RemovePersonFromGroupCommand parse(String args) throws ParseException {
        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_GROUP, PREFIX_PERSON);

        if (!arePrefixesPresent(argMultimap, PREFIX_GROUP, PREFIX_PERSON)) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT,
                    AddPersonToGroupCommand.MESSAGE_USAGE));
        }

        try {
            Index groupIndex = ParserUtil.parseIndex(argMultimap.getValue(PREFIX_GROUP).get());
            Index personIndex = ParserUtil.parseIndex(argMultimap.getValue(PREFIX_PERSON).get());
            return new RemovePersonFromGroupCommand(groupIndex, personIndex);
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
###### \java\seedu\address\logic\parser\SelectCommandParser.java
``` java
    /**
     * Parses the given {@code String} of arguments in the context of the SelectCommand
     * and returns an SelectCommand object for execution.
     *
     * @throws ParseException if the user input does not conform the expected format
     */
    public SelectCommand parse(String args) throws ParseException {
        if (!args.matches("^( (g/)?[\\d]+)$")) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, SelectCommand.MESSAGE_USAGE));
        }

        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_GROUP);
        try {
            if (args.trim().contains(PREFIX_GROUP.getPrefix())) {
                isGroup = true;
                index = ParserUtil.parseIndex(argMultimap.getValue(PREFIX_GROUP).get());
            } else {
                index = ParserUtil.parseIndex(args);
            }
            return new SelectCommand(index, isGroup);
        } catch (IllegalValueException ive) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, SelectCommand.MESSAGE_USAGE));
        }

    }

}
```
###### \java\seedu\address\logic\parser\SortCommandParser.java
``` java
/**
 * Parses input arguments and creates a new SortCommand object
 */
public class SortCommandParser implements Parser<SortCommand> {

    private String field;
    private Boolean isReverseOrder = false;

    /**
     * Parses the given {@code String} of arguments in the context of the SortCommand
     * and returns an SortCommand object for execution.
     * @throws ParseException if the user input does not conform the expected format
     */

    public SortCommand parse(String args) throws ParseException {
        String args2 = args; // for codacy issue fix to prevent original arg overwrite
        requireNonNull(args2);

        if (!args.matches("^|( [npea]/(r)?)$")) {
            throw new ParseException(String.format(MESSAGE_INVALID_COMMAND_FORMAT, SortCommand.MESSAGE_USAGE));
        }

        if ("".equals(args)) {
            args = " n/";
        }

        ArgumentMultimap argMultimap =
                ArgumentTokenizer.tokenize(args, PREFIX_NAME, PREFIX_PHONE, PREFIX_EMAIL, PREFIX_ADDRESS);

        argMultimap.getValue(PREFIX_NAME).ifPresent(setOrder(PREFIX_NAME));
        argMultimap.getValue(PREFIX_PHONE).ifPresent(setOrder(PREFIX_PHONE));
        argMultimap.getValue(PREFIX_EMAIL).ifPresent(setOrder(PREFIX_EMAIL));
        argMultimap.getValue(PREFIX_ADDRESS).ifPresent(setOrder(PREFIX_ADDRESS));

        return new SortCommand(field, isReverseOrder);

    }

    private Consumer<String> setOrder(Prefix prefix) {
        return s -> {

            field = prefix.toString();

            if (s.equals(SortCommand.REVERSE_ORDER)) {
                isReverseOrder = Boolean.TRUE;
                return;
            } else {
                isReverseOrder = Boolean.FALSE;
                return;

            }
        };

    }


}
```
###### \java\seedu\address\model\AddressBook.java
``` java
    /**
     * Replaces the given person {@code target} in the list with {@code editedFavouritePerson}.
     * {@code AddressBook}'s tag list will be updated with the tags of {@code editedFavouritePerson}.
     *
     * @throws DuplicatePersonException if updating the person's details causes the person to be equivalent to
     *      another existing person in the list.
     * @throws PersonNotFoundException if {@code target} could not be found in the list.
     *
     * @see #syncMasterTagListWith(Person)
     */
    public void updateFavouriteStatus(ReadOnlyPerson target, ReadOnlyPerson editedFavouritePerson)
            throws DuplicatePersonException, PersonNotFoundException {
        requireNonNull(editedFavouritePerson);
        Person editedPerson = new Person(editedFavouritePerson);
        syncMasterTagListWith(editedPerson);
        syncMasterGroupListWith(editedPerson);
        persons.setFavourite(target, editedPerson);
    }


    /**
     * Sorts persons in address book.
     */

    public void sortPerson(Comparator<ReadOnlyPerson> sortComparator, boolean isReverseOrder)
            throws NoPersonsException {
        persons.sort(sortComparator, isReverseOrder);
    }

```
###### \java\seedu\address\model\AddressBook.java
``` java
    //// group-level operations

    /**
     * Adds a group to the address book.
     *
     * @throws DuplicateGroupException if an equivalent group already exists.
     */

    public void addGroup(ReadOnlyGroup g) throws DuplicateGroupException {
        Group newGroup = new Group(g);
        groups.add(newGroup);
    }

    /**
     * Adds a person to a group in the address book.
     *
     * @throws GroupNotFoundException if group does not exist.
     * @throws PersonNotFoundException if person does not exist.
     * @throws DuplicatePersonException if an equivalent person already exists.
     *
     */

    public void addPersonToGroup(Index targetGroup, ReadOnlyPerson toAdd)
            throws GroupNotFoundException, PersonNotFoundException, DuplicatePersonException {
        groups.addPersonToGroup(targetGroup, toAdd);
    }

    /**
     * Adds a person to a group in the address book.
     *
     * @throws GroupNotFoundException if group does not exist.
     * @throws PersonNotFoundException if person does not exist.
     * @throws NoPersonsException if group is empty.
     *
     */

    public void deletePersonFromGroup(Index targetGroup, ReadOnlyPerson toAdd)
            throws GroupNotFoundException, PersonNotFoundException, NoPersonsException {
        groups.removePersonFromGroup(targetGroup, toAdd);
    }

    /**
     * Removes {@code key} from this {@code AddressBook}.
     * @throws GroupNotFoundException if the {@code key} is not in this {@code AddressBook}.
     */
    public boolean removeGroup(ReadOnlyGroup key) throws GroupNotFoundException {
        if (groups.remove(key)) {
            return true;
        } else {
            throw new GroupNotFoundException();
        }
    }

```
###### \java\seedu\address\model\group\exceptions\DuplicateGroupException.java
``` java
/**
 * Signals that the operation will result in duplicate Person objects.
 */
public class DuplicateGroupException extends DuplicateDataException {
    public DuplicateGroupException() {
        super("Operation would result in duplicate groups");
    }
}
```
###### \java\seedu\address\model\group\exceptions\GroupNotFoundException.java
``` java
/**
 * Signals that the operation is unable to find the specified person.
 */
public class GroupNotFoundException extends Exception {}
```
###### \java\seedu\address\model\group\exceptions\NoGroupsException.java
``` java
/**
 * Signals that the operation is unable to sort due to an empty list.
 */
public class NoGroupsException extends Exception {}
```
###### \java\seedu\address\model\group\Group.java
``` java
/**
 * Represents a Group in an address book.
 * Guarantees: details are present and not null, field values are validated.
 */
public class Group implements ReadOnlyGroup {

    private ObjectProperty<GroupName> groupName;
    /**
     * A Group will have an empty persons list by default
     */
    private ObjectProperty<UniquePersonList> groupMembers =
            new SimpleObjectProperty<>(new UniquePersonList());

    /**
     * Every field must be present and not null.
     */
    public Group(GroupName name) {
        requireNonNull(name);
        this.groupName = new SimpleObjectProperty<>(name);
    }

    /**
     * Every field must be present and not null.
     */
    public Group(GroupName name, Set<Person> groupMembers) {
        requireAllNonNull(name, groupMembers);
        this.groupName = new SimpleObjectProperty<>(name);
        this.groupMembers = new SimpleObjectProperty<>(new UniquePersonList(groupMembers));
    }

    /**
     * Every field must be present and not null.
     */
    public Group(String name) throws IllegalValueException {
        requireNonNull(name);
        this.groupName = new SimpleObjectProperty<>(new GroupName(name));
    }

    /**
     * Creates a copy of the given ReadOnlyGroup.
     */
    public Group(ReadOnlyGroup source) {
        this(source.getName(), source.getMembers());
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof Group // instanceof handles nulls
                && this.groupName.toString().equals(((Group) other).groupName.toString())); // state check
    }

    @Override
    public int hashCode() {
        return groupName.hashCode();
    }

    /**
     * Format state as text for viewing.
     */
    public String toString() {
        return getAsText();
    }

    public void addMember(ReadOnlyPerson person) throws DuplicatePersonException {
        this.groupMembers.get().add(person);
    }

    public void deleteMember(ReadOnlyPerson person) throws PersonNotFoundException {
        this.groupMembers.get().remove(person);
    }

    @Override
    public ObjectProperty<GroupName> nameProperty() {
        return groupName;
    }

    @Override
    public GroupName getName() {
        return groupName.get();
    }

    @Override
    public ObjectProperty<UniquePersonList> groupMembersProperty() {
        return groupMembers;
    }

    public void setGroupName(GroupName name) {
        this.groupName.set(requireNonNull(name));
    }

    @Override
    public Set<Person> getMembers() {
        return groupMembers.get().toSet();
    }


}
```
###### \java\seedu\address\model\group\GroupName.java
``` java
/**
 * Represents a Person's name in the address book.
 * Guarantees: immutable; is valid as declared in {@link #isValidName(String)}
 */
public class GroupName {

    public static final String MESSAGE_GROUP_CONSTRAINTS = "Group names should contain only "
            + "alphanumeric characters, spaces, underscores and dashes";

    /*
     * The first character of the address must not be a whitespace,
     * otherwise " " (a blank string) becomes a valid input.
     */
    public static final String GROUP_VALIDATION_REGEX = "^[a-zA-Z0-9]([\\w -]*[a-zA-Z0-9])?$";

    public final String fullName;

    /**
     * Validates given name.
     *
     * @throws IllegalValueException if given name string is invalid.
     */
    public GroupName(String name) throws IllegalValueException {
        requireNonNull(name);
        String trimmedName = name.trim();
        if (!isValidName(trimmedName)) {
            throw new IllegalValueException(MESSAGE_GROUP_CONSTRAINTS);
        }
        this.fullName = trimmedName;
    }

    /**
     * Returns true if a given string is a valid person name.
     */
    public static boolean isValidName(String test) {
        return test.matches(GROUP_VALIDATION_REGEX);
    }


    @Override
    public String toString() {
        return fullName;
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof GroupName // instanceof handles nulls
                && this.fullName.equals(((GroupName) other).fullName)); // state check
    }

    @Override
    public int hashCode() {
        return fullName.hashCode();
    }

}
```
###### \java\seedu\address\model\group\ReadOnlyGroup.java
``` java
/**
 * A read-only immutable interface for a Group in the addressbook.
 * Implementations should guarantee: details are present and not null, field values are validated.
 */
public interface ReadOnlyGroup {

    ObjectProperty<GroupName> nameProperty();
    GroupName getName();
    ObjectProperty<UniquePersonList> groupMembersProperty();
    Set<Person> getMembers();

    /**
     * Returns true if both have the same state. (interfaces cannot override .equals)
     */
    default boolean isSameStateAs(ReadOnlyGroup other) {
        return other == this // short circuit if same object
                || (other != null // this is first to avoid NPE below
                && other.getName().equals(this.getName()) // state checks here onwards
                && other.getMembers().equals(this.getMembers()));
    }

    /**
     * Formats the Group as text, showing group name.
     */
    default String getAsText() {
        final StringBuilder builder = new StringBuilder();
        builder.append(getName())
                .append(" Group Name: ")
                .append(getName());
        return builder.toString();
    }

}
```
###### \java\seedu\address\model\group\UniqueGroupList.java
``` java
/**
 * A list of groups that enforces no nulls and uniqueness between its elements.
 *
 * Supports minimal set of list operations for the app's features.
 *
 * @see Group#equals(Object)
 */


public class UniqueGroupList implements Iterable<Group> {

    private final ObservableList<Group> internalList = FXCollections.observableArrayList();
    // used by asObservableList()
    private final ObservableList<ReadOnlyGroup> mappedList = EasyBind.map(internalList, (group) -> group);

    /**
     * Creates a UniqueGroupList using given Groups.
     * Enforces no nulls.
     */
    public UniqueGroupList(Set<Group> groups) {
        requireAllNonNull(groups);
        internalList.addAll(groups);

        assert CollectionUtil.elementsAreUnique(internalList);
    }

    /**
     * Constructs empty UniqueGroupList.
     */
    public UniqueGroupList() {}

    /**
     * Returns true if the list contains an equivalent Group as the given argument.
     */
    public boolean contains(ReadOnlyGroup toCheck) {
        requireNonNull(toCheck);
        return internalList.contains(toCheck);
    }

    /**
     * Returns all groups in this list as a Set.
     * This set is mutable and change-insulated against the internal list.
     */
    public Set<Group> toSet() {
        assert CollectionUtil.elementsAreUnique(internalList);
        return new HashSet<>(internalList);
    }

    /**
     * Ensures every group in the argument list exists in this object.
     */
    public void mergeFrom(UniqueGroupList from) {
        final Set<Group> alreadyInside = this.toSet();
        from.internalList.stream()
                .filter(group -> !alreadyInside.contains(group))
                .forEach(internalList::add);

        assert CollectionUtil.elementsAreUnique(internalList);
    }

    /**
     * Adds a Group to the list.
     *
     * @throws seedu.address.model.group.exceptions.DuplicateGroupException
     * if the Tag to add is a duplicate of an existing Tag in the list.
     */
    public void add(ReadOnlyGroup toAdd) throws DuplicateGroupException {
        requireNonNull(toAdd);
        if (contains(toAdd)) {
            throw new DuplicateGroupException();
        }
        internalList.add(new Group(toAdd));

        assert CollectionUtil.elementsAreUnique(internalList);
    }

    /**
     * Removes the equivalent person from the list.
     *
     * @throws GroupNotFoundException if no such group could be found in the list.
     */
    public boolean remove(ReadOnlyGroup toRemove) throws GroupNotFoundException {
        requireNonNull(toRemove);
        final boolean groupFoundAndDeleted = internalList.remove(toRemove);
        if (!groupFoundAndDeleted) {
            throw new GroupNotFoundException();
        }
        return groupFoundAndDeleted;
    }

    /**
     * Adds person to specified group in the list.
     *
     * @throws GroupNotFoundException if no such group could be found in the list.
     * @throws DuplicatePersonException if an equivalent person exists in the list.
     * @throws PersonNotFoundException if no such person could be found in the list.
     */
    public void addPersonToGroup(Index target, ReadOnlyPerson toAdd)
            throws GroupNotFoundException, DuplicatePersonException, PersonNotFoundException {
        requireNonNull(toAdd);
        requireNonNull(target);

        Group targetGroup = internalList.get(target.getZeroBased());

        if (isNull(targetGroup)) {
            throw new GroupNotFoundException();
        }

        try {
            targetGroup.addMember(toAdd);
        } catch (DuplicatePersonException dpe) {
            throw new DuplicatePersonException();
        }

        internalList.set(target.getZeroBased(), targetGroup);
    }

    /**
     * Removes person from specified group in the list.
     *
     * @throws GroupNotFoundException if no such group could be found in the list.
     * @throws NoPersonsException if list is empty.
     * @throws PersonNotFoundException if no such person could be found in the list.
     */
    public void removePersonFromGroup(Index target, ReadOnlyPerson toAdd)
            throws GroupNotFoundException, NoPersonsException, PersonNotFoundException {
        requireNonNull(toAdd);
        requireNonNull(target);

        Group targetGroup = internalList.get(target.getZeroBased());

        if (targetGroup.getMembers().size() < 1) {
            throw new NoPersonsException();
        }

        if (isNull(targetGroup)) {
            throw new GroupNotFoundException();
        }

        try {
            targetGroup.deleteMember(toAdd);
        } catch (PersonNotFoundException pnfe) {
            throw new PersonNotFoundException();
        }

        internalList.set(target.getZeroBased(), targetGroup);
    }

    @Override
    public Iterator<Group> iterator() {
        assert CollectionUtil.elementsAreUnique(internalList);
        return internalList.iterator();
    }


    public void setGroups(UniqueGroupList replacement) {
        this.internalList.setAll(replacement.internalList);
    }

    public void setGroups(List<? extends ReadOnlyGroup> groups) throws DuplicateGroupException {
        final UniqueGroupList replacement = new UniqueGroupList();
        for (final ReadOnlyGroup group : groups) {
            replacement.add(new Group(group));
        }
        setGroups(replacement);
    }


    /**
     * Returns the backing list as an unmodifiable {@code ObservableList}.
     */
    public ObservableList<ReadOnlyGroup> asObservableList() {
        assert CollectionUtil.elementsAreUnique(internalList);
        return FXCollections.unmodifiableObservableList(mappedList);
    }

    @Override
    public boolean equals(Object other) {
        assert CollectionUtil.elementsAreUnique(internalList);
        return other == this // short circuit if same object
                || (other instanceof seedu.address.model.group.UniqueGroupList // instanceof handles nulls
                && this.internalList.equals(((seedu.address.model.group.UniqueGroupList) other).internalList));
    }

    /**
     * Returns true if the element in this list is equal to the elements in {@code other}.
     * The elements do not have to be in the same order.
     */
    public boolean equalsOrderInsensitive(seedu.address.model.group.UniqueGroupList other) {
        assert CollectionUtil.elementsAreUnique(internalList);
        assert CollectionUtil.elementsAreUnique(other.internalList);
        return this == other || new HashSet<>(this.internalList).equals(new HashSet<>(other.internalList));
    }

    @Override
    public int hashCode() {
        assert CollectionUtil.elementsAreUnique(internalList);
        return internalList.hashCode();
    }

}

```
###### \java\seedu\address\model\Model.java
``` java
    /** Sorts address book list */
    void sortPerson(Comparator<ReadOnlyPerson> sortComparator, boolean isReverseOrder) throws NoPersonsException;

    /** Adds the given group */
    void addGroup(ReadOnlyGroup group) throws DuplicateGroupException;

    /** Deletes the given group */
    void deleteGroup(ReadOnlyGroup group) throws GroupNotFoundException;

    void addPersonToGroup(Index targetGroup, ReadOnlyPerson toAdd) throws
            GroupNotFoundException, PersonNotFoundException, DuplicatePersonException;

    /** Deletes given person from given group */
    void deletePersonFromGroup(Index targetGroup, ReadOnlyPerson toRemove) throws
            GroupNotFoundException, PersonNotFoundException, NoPersonsException;

```
###### \java\seedu\address\model\Model.java
``` java
    /**
     * Replaces the given person {@code target} with {@code favouritePerson}.
     *
     * @throws DuplicatePersonException if updating the person's details causes the person to be equivalent to
     *      another existing person in the list.
     * @throws PersonNotFoundException if {@code target} could not be found in the list.
     */
    void updateFavouritePerson(ReadOnlyPerson target, ReadOnlyPerson favouritePerson)
            throws DuplicatePersonException, PersonNotFoundException;

```
###### \java\seedu\address\model\ModelManager.java
``` java
    @Override
    public void updateFavouritePerson(ReadOnlyPerson target, ReadOnlyPerson favouritePerson)
            throws DuplicatePersonException, PersonNotFoundException {
        requireAllNonNull(target, favouritePerson);
        addressBook.updateFavouriteStatus(target, favouritePerson);
        indicateAddressBookChanged();
    }

    @Override
    public void sortPerson(Comparator<ReadOnlyPerson> sortComparator, boolean isReverseOrder)
            throws NoPersonsException {
        addressBook.sortPerson(sortComparator, isReverseOrder);
    }

    @Override
    public void addGroup(ReadOnlyGroup group) throws DuplicateGroupException {
        addressBook.addGroup(group);
        updateFilteredGroupList(PREDICATE_SHOW_ALL_GROUPS);
        indicateAddressBookChanged();
    }

    @Override
    public void deleteGroup(ReadOnlyGroup target) throws GroupNotFoundException {
        addressBook.removeGroup(target);
        indicateAddressBookChanged();
    }

    @Override
    public void addPersonToGroup(Index targetGroup, ReadOnlyPerson toAdd)
            throws GroupNotFoundException, PersonNotFoundException, DuplicatePersonException {
        addressBook.addPersonToGroup(targetGroup, toAdd);
        indicateAddressBookChanged();
    }

    @Override
    public void deletePersonFromGroup(Index targetGroup, ReadOnlyPerson toRemove)
            throws GroupNotFoundException, PersonNotFoundException, NoPersonsException {
        addressBook.deletePersonFromGroup(targetGroup, toRemove);
        /** Update filtered list with predicate for current group members in group after removing a person */
        ObservableList<ReadOnlyPerson> personList = addressBook.getGroupList()
                .get(targetGroup.getZeroBased()).groupMembersProperty().get().asObservableList();
        updateFilteredPersonList(getGroupMembersPredicate(personList));
        indicateAddressBookChanged();

    }
```
###### \java\seedu\address\model\ModelManager.java
``` java
    /** Returns predicate that returns true if group member list contains a person */
    /** Used to update FilteredPersonList whenever there is a need to display group members */
    public Predicate<ReadOnlyPerson> getGroupMembersPredicate(ObservableList<ReadOnlyPerson> personList) {
        return personList::contains;
    }

    /** Handle any GroupPanelSelectionChangedEvent raised and set predicate to show group members only */
    @Subscribe
    private void handleGroupPanelSelectionChangedEvent(GroupPanelSelectionChangedEvent event) {
        ObservableList<ReadOnlyPerson> personList = event.getNewSelection()
                .group.groupMembersProperty().get().asObservableList();
        logger.info(LogsCenter.getEventHandlingLogMessage(event));
        updateFilteredPersonList(getGroupMembersPredicate(personList));
    }
```
###### \java\seedu\address\model\person\Person.java
``` java
    /**
     * Every field must be present and not null.
     * Constructor for Schedule feature
     */
    public Person(Name name, Phone phone, Email email, Address address, Favourite favourite,
                  Set<Tag> tags, Set<Schedule> schedule) {
        requireAllNonNull(name, phone, email, address, favourite, tags, schedule);
        this.name = new SimpleObjectProperty<>(name);
        this.phone = new SimpleObjectProperty<>(phone);
        this.email = new SimpleObjectProperty<>(email);
        this.address = new SimpleObjectProperty<>(address);
        this.favourite = new SimpleObjectProperty<>(favourite);
        this.tags = new SimpleObjectProperty<>(new UniqueTagList(tags));
        this.schedule = new SimpleObjectProperty<>(new UniqueScheduleList(schedule));
    }

```
###### \java\seedu\address\model\person\Person.java
``` java
    /**
     * Every field must be present and not null.
     * Constructor for Group feature
     */
    public Person(Name name, Phone phone, Email email, Address address, Set<Tag> tags, Set<Group> groups) {
        requireAllNonNull(name, phone, email, address, tags, groups);
        this.name = new SimpleObjectProperty<>(name);
        this.phone = new SimpleObjectProperty<>(phone);
        this.email = new SimpleObjectProperty<>(email);
        this.address = new SimpleObjectProperty<>(address);
        // protect internal tags from changes in the arg list
        this.tags = new SimpleObjectProperty<>(new UniqueTagList(tags));
        // protect internal groups from changes in the arg list
        this.groups = new SimpleObjectProperty<>(new UniqueGroupList(groups));
    }

```
###### \java\seedu\address\model\person\UniquePersonList.java
``` java
    /**
     * Sorts persons in address book by field and in order specified.
     * @param sortComparator
     * @param isReverseOrder
     * @throws NoPersonsException
     */

    public void sort(Comparator sortComparator, Boolean isReverseOrder) throws NoPersonsException {
        requireNonNull(sortComparator);
        requireNonNull(isReverseOrder);

        if (internalList.size() < 1) {
            throw new NoPersonsException();
        }

        Collections.sort(internalList, sortComparator);

        if (isReverseOrder) {
            Collections.reverse(internalList);
        }
    }

```
###### \java\seedu\address\model\person\UniquePersonList.java
``` java
    /**
     * Replaces the person {@code target} in the list with {@code favouritePerson}.
     * @throws DuplicatePersonException if the replacement is equivalent to another existing person in the list.
     * @throws PersonNotFoundException if {@code target} could not be found in the list.
     */
    public void setFavourite(ReadOnlyPerson target, ReadOnlyPerson favouritePerson)
            throws DuplicatePersonException, PersonNotFoundException {
        requireNonNull(favouritePerson);

        int index = internalList.indexOf(target);
        if (index == -1) {
            throw new PersonNotFoundException();
        }

        if (!target.equals(favouritePerson) && internalList.contains(favouritePerson)) {
            throw new DuplicatePersonException();
        }

        int targetIndex;

        /** Main favourite (fadd) logic
         * If person is marked as favourite, remove it from its current position
         * Attempt to insert at the head of the list
         * If the person at the top is favourite
         * Find new position where:
         * It's lexicographically smaller than the person's name preceding it
         * i.e If the edited person's name is Ben, it should come after Alex, if Alex is marked as favourite
         * Insert at the new position
         */

        if (favouritePerson.getFavourite().getStatus()) {
            targetIndex = 0;
            ReadOnlyPerson currentPerson;
            for (int i = 0; i < internalList.size(); i++) {
                currentPerson = internalList.get(i);
                if (currentPerson.getFavourite().getStatus()) {
                    if (currentPerson.getName().fullName.compareTo(favouritePerson.getName().fullName) < 0) {
                        targetIndex++;
                    }
                }
            }

            internalList.remove(index);
            internalList.add(targetIndex, new Person(favouritePerson));


        } else {

            /** Main favourite (fremove) logic
             * If person is unmarked as favourite, insert person at new position
             * Find new position where:
             * New position should be after all the favourites
             * Insert at the new position
             */

            targetIndex = index;
            for (int i = index + 1; i < internalList.size(); i++) {
                if (internalList.get(i).getFavourite().getStatus()) {
                    targetIndex++;
                }
            }

            /** If there is no change in position, do not remove person
             *  Continue with normal edit logic
             */
            if (targetIndex != index) {
                internalList.remove(index);
                internalList.add(targetIndex, new Person(favouritePerson));
            } else {
                internalList.set(index, new Person(favouritePerson));
            }
        }
    }



    /**
     * Removes the equivalent person from the list.
     *
     * @throws PersonNotFoundException if no such person could be found in the list.
     */
    public boolean remove(ReadOnlyPerson toRemove) throws PersonNotFoundException {
        requireNonNull(toRemove);
        final boolean personFoundAndDeleted = internalList.remove(toRemove);
        if (!personFoundAndDeleted) {
            throw new PersonNotFoundException();
        }
        return personFoundAndDeleted;
    }

    public void setPersons(UniquePersonList replacement) {
        this.internalList.setAll(replacement.internalList);
    }

    public void setPersons(List<? extends ReadOnlyPerson> persons) throws DuplicatePersonException {
        final UniquePersonList replacement = new UniquePersonList();
        for (final ReadOnlyPerson person : persons) {
            replacement.add(new Person(person));
        }
        setPersons(replacement);
    }

    /**
     * Returns the backing list as an unmodifiable {@code ObservableList}.
     */
    public ObservableList<ReadOnlyPerson> asObservableList() {
        return FXCollections.unmodifiableObservableList(mappedList);
    }

    @Override
    public Iterator<Person> iterator() {
        return internalList.iterator();
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof UniquePersonList // instanceof handles nulls
                        && this.internalList.equals(((UniquePersonList) other).internalList));
    }

    @Override
    public int hashCode() {
        return internalList.hashCode();
    }

    /**
     * Returns all persons in this list as a Set.
     * This set is mutable and change-insulated against the internal list.
     */
    public Set<Person> toSet() {
        assert CollectionUtil.elementsAreUnique(internalList);
        return new HashSet<>(internalList);
    }
}
```
###### \java\seedu\address\model\schedule\ReadOnlySchedule.java
``` java
/**
 * A read-only immutable interface for a Schedule in the addressbook.
 * Implementations should guarantee: details are present and not null, field values are validated.
 */
public interface ReadOnlySchedule {

    ObjectProperty<ScheduleName> nameProperty();
    ScheduleName getName();
    ObjectProperty<ScheduleDate> startDateTimeProperty();
    ScheduleDate getStartDateTime();
    ObjectProperty<ScheduleDate> endDateTimeProperty();
    ScheduleDate getEndDateTime();
    String getScheduleDuration();
    SimpleStringProperty scheduleDurationProperty();
    String getScheduleDetails();
    SimpleStringProperty scheduleDetailsProperty();



    /**
     * Returns true if both have the same state. (interfaces cannot override .equals)
     */
    default boolean isSameStateAs(ReadOnlySchedule other) {
        return other == this // short circuit if same object
                || (other != null // this is first to avoid NPE below
                && other.getName().equals(this.getName())
                && other.getStartDateTime().equals(this.getStartDateTime())
                && other.getEndDateTime().equals(this.getEndDateTime())
                && other.getScheduleDuration().equals(this.getScheduleDuration())); // state checks here onwards
    }

    /**
     * Formats the Schedule as text, showing schedule name.
     */
    default String getAsText() {
        final StringBuilder builder = new StringBuilder();
        builder.append(getName())
                .append(" Schedule Name: ")
                .append(getName());
        return builder.toString();
    }

}
```
###### \java\seedu\address\model\schedule\Schedule.java
``` java
/**
 * Represents a Schedule in an address book.
 * Guarantees: details are present and not null, field values are validated.
 */
public class Schedule implements ReadOnlySchedule {

    private ObjectProperty<ScheduleName> scheduleName;
    private ObjectProperty<ScheduleDate> startDateTime;
    private ObjectProperty<ScheduleDate> endDateTime;
    private SimpleStringProperty scheduleDuration;
    private SimpleStringProperty scheduleDetails = new SimpleStringProperty();

    /**
     * Every field must be present and not null.
     */
    public Schedule(ScheduleName name, ScheduleDate startDateTime, ScheduleDate endDateTime,
                    String scheduleDuration, String scheduleDetails) {
        requireNonNull(name);
        this.scheduleName = new SimpleObjectProperty<>(name);
        this.startDateTime = new SimpleObjectProperty<>(startDateTime);
        this.endDateTime = new SimpleObjectProperty<>(endDateTime);
        this.scheduleDuration = new SimpleStringProperty(scheduleDuration);
        this.scheduleDetails = new SimpleStringProperty(scheduleDetails);
    }

    /**
     * Every field must be present and not null.
     */
    public Schedule(String name) throws IllegalValueException {
        requireNonNull(name);
        this.scheduleName = new SimpleObjectProperty<>(new ScheduleName(name));
    }
    /**
     * Creates a copy of the given ReadOnlySchedule.
     */
    public Schedule(ReadOnlySchedule source) {
        this(source.getName(), source.getStartDateTime(), source.getEndDateTime(),
                source.getScheduleDuration(), source.getScheduleDetails());
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof Schedule // instanceof handles nulls
                && this.scheduleName.toString().equals(((Schedule) other).scheduleName.toString())
                && this.startDateTime.toString().equals(((Schedule) other).startDateTime.toString())
                && this.endDateTime.toString().equals(((Schedule) other).endDateTime.toString())); // state check
    }

    @Override
    public int hashCode() {
        return scheduleName.hashCode();
    }

    /**
     * Format state as text for viewing.
     */
    public String toString() {
        return getAsText();
    }


    @Override
    public ObjectProperty<ScheduleName> nameProperty() {
        return scheduleName;
    }

    @Override
    public ScheduleName getName() {
        return scheduleName.get();
    }

    @Override
    public ObjectProperty<ScheduleDate> startDateTimeProperty() {
        return startDateTime;
    }

    @Override
    public ScheduleDate getStartDateTime() {
        return startDateTime.get();
    }

    @Override
    public ObjectProperty<ScheduleDate> endDateTimeProperty() {
        return endDateTime;
    }

    @Override
    public ScheduleDate getEndDateTime() {
        return endDateTime.get();
    }

    public void setScheduleName(ScheduleName name) {
        this.scheduleName.set(requireNonNull(name));
    }

    @Override
    public String getScheduleDuration() {
        return scheduleDuration.get();
    }

    @Override
    public SimpleStringProperty scheduleDurationProperty() {
        return scheduleDuration;
    }

    @Override
    public String getScheduleDetails() {
        return scheduleDetails.get();
    }

    @Override
    public SimpleStringProperty scheduleDetailsProperty() {
        return scheduleDetails;
    }

}
```
###### \java\seedu\address\model\schedule\ScheduleDate.java
``` java
/**
 * Represents a Schedule's date string in the address book.
 * Guarantees: immutable; Valid schedule date
 */
public class ScheduleDate {

    public static final String MESSAGE_SCHEDULE_DATE_CONSTRAINTS =
            "Schedule date should be in the following format: "
            + "YYYY-MM-DD HH:MM";


    public final String scheduleDateString;
    public final Date scheduleDate;

    /**
     * Validates given name.
     *
     * @throws ParseException if given name string is invalid.
     */
    public ScheduleDate(String date) throws ParseException {
        requireNonNull(date);
        DateFormat dateInput = new SimpleDateFormat("yyyy-MM-dd hh:mm");

        try {
            scheduleDate = dateInput.parse(date);
        } catch (java.text.ParseException e) {
            throw new ParseException(MESSAGE_SCHEDULE_DATE_CONSTRAINTS);
        }

        this.scheduleDateString = date;

    }

    @Override
    public String toString() {
        return scheduleDateString;
    }

    public Date getScheduleDate() {
        return scheduleDate; }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof ScheduleDate // instanceof handles nulls
                && this.scheduleDate.equals(((ScheduleDate) other).scheduleDate)); // state check
    }

    @Override
    public int hashCode() {
        return scheduleDate.hashCode();
    }

}
```
###### \java\seedu\address\model\schedule\ScheduleName.java
``` java
/**
 * Represents a Schedule's name in the address book.
 */
public class ScheduleName {

    public final String fullName;

    /**
     * Constructs a ScheduleName object
     */
    public ScheduleName(String name) {
        requireNonNull(name);
        String trimmedName = name.trim();
        this.fullName = trimmedName;
    }

    @Override
    public String toString() {
        return fullName;
    }

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof ScheduleName // instanceof handles nulls
                && this.fullName.equals(((ScheduleName) other).fullName)); // state check
    }

    @Override
    public int hashCode() {
        return fullName.hashCode();
    }

}
```
###### \java\seedu\address\model\schedule\UniqueScheduleList.java
``` java
    /** Sorts scheduleList by schedule end dateTime **/
    public void sort() {
        Collections.sort(internalList, (s1, s2) -> {
            if (s1.getEndDateTime().scheduleDate.getTime() < s2.getEndDateTime().scheduleDate.getTime()) {
                return -1;
            } else if (s1.getEndDateTime().scheduleDate.getTime() == s2.getEndDateTime().scheduleDate.getTime()) {
                return 0;
            } else {
                return 1;
            }
        });

    }

```
###### \java\seedu\address\storage\XmlAdaptedGroup.java
``` java
/**
 * JAXB-friendly adapted version of the Tag.
 */
public class XmlAdaptedGroup {

    @XmlElement(required = true)
    private String groupName;
    @XmlElement
    private List<XmlAdaptedPerson> members = new ArrayList<>();
    /**
     * Constructs an XmlAdaptedGroup.
     * This is the no-arg constructor that is required by JAXB.
     */
    public XmlAdaptedGroup() {}

    /**
     * Converts a given Tag into this class for JAXB use.
     *
     * @param source future changes to this will not affect the created
     */
    public XmlAdaptedGroup(ReadOnlyGroup source) {
        groupName = source.getName().toString();
        for (ReadOnlyPerson person: source.getMembers()) {
            members.add(new XmlAdaptedPerson(person));
        }

    }

    /**
     * Converts this jaxb-friendly adapted group object into the model's Group object.
     *
     * @throws IllegalValueException if there were any data constraints violated in the adapted person
     */
    public Group toModelType() throws IllegalValueException {
        final List<Person> personList = new ArrayList<>();
        for (XmlAdaptedPerson person: members) {
            personList.add(person.toModelType());
        }

        final GroupName groupName = new GroupName(this.groupName);
        final Set<Person> persons = new HashSet<>(personList);
        return new Group(groupName, persons);
    }

}
```
###### \java\seedu\address\ui\PersonCard.java
``` java
        favicon.visibleProperty().bind(Bindings.createBooleanBinding(() -> person.getFavourite().value));
```
