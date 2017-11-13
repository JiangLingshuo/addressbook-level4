# Procrastinatus
###### \java\seedu\address\commons\events\ui\JumpToScheduleListRequestEvent.java
``` java
/**
 * Indicates a request to jump to the list of persons
 */
public class JumpToScheduleListRequestEvent extends BaseEvent {

    public final int targetIndex;

    public JumpToScheduleListRequestEvent(Index targetIndex) {
        this.targetIndex = targetIndex.getZeroBased();
    }

    @Override
    public String toString() {
        return this.getClass().getSimpleName();
    }

}
```
###### \java\seedu\address\commons\events\ui\SchedulePanelSelectionChangedEvent.java
``` java
/**
 * Represents a selection change in the Person List Panel
 */
public class SchedulePanelSelectionChangedEvent extends BaseEvent {

    private final ScheduleCard newSelection;

    public SchedulePanelSelectionChangedEvent(ScheduleCard newSelection) {
        this.newSelection = newSelection;
    }

    @Override
    public String toString() {
        return this.getClass().getSimpleName();
    }

    public ScheduleCard getNewSelection() {
        return newSelection;
    }
}
```
###### \java\seedu\address\logic\commands\DeleteCommand.java
``` java
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Deletes the person identified by the index number(s) used in the last person listing.\n"
            + "Parameters: INDEX (must be a positive integer)\n"
            + "Example: " + COMMAND_WORD + " 1 2 3";

    public static final String MESSAGE_DELETE_PERSON_SUCCESS = "Deleted Person: %1$s";

    private Index targetIndex;

    private Index[] targetIndexes;

    public DeleteCommand(Index targetIndex) {
        this.targetIndex = targetIndex;
    }

    public DeleteCommand(Index[] targetIndexes) {
        this.targetIndexes = targetIndexes;
    }

    @Override
    public CommandResult executeUndoableCommand() throws CommandException {

        List<ReadOnlyPerson> lastShownList = model.getFilteredPersonList();

        ReadOnlyPerson personToDelete = null;
        ReadOnlyPerson[] personsToDelete = null;

        if (targetIndex != null) {
            if (targetIndex.getZeroBased() >= lastShownList.size()) {
                throw new CommandException(Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
            }
            personToDelete = lastShownList.get(targetIndex.getZeroBased());
        }

        if (targetIndexes != null) {
            personsToDelete = new ReadOnlyPerson[targetIndexes.length];
            for (int i = 0; i < personsToDelete.length; i++) {
                personsToDelete[i] = lastShownList.get(targetIndexes[i].getZeroBased());
            }
        }

        try {
            if (personsToDelete == null) {
                model.deletePerson(personToDelete);
            } else {
                model.deletePersons(personsToDelete);
            }
        } catch (PersonNotFoundException pnfe) {
            assert false : "The target person cannot be missing";
        }

        return new CommandResult(String.format(MESSAGE_DELETE_PERSON_SUCCESS, personToDelete));
    }
```
###### \java\seedu\address\logic\commands\DeleteCommand.java
``` java

    @Override
    public boolean equals(Object other) {
        return other == this // short circuit if same object
                || (other instanceof DeleteCommand // instanceof handles nulls
                && this.targetIndex.equals(((DeleteCommand) other).targetIndex)); // state check
    }
}
```
###### \java\seedu\address\logic\parser\CliSyntax.java
``` java
    public static final Prefix PREFIX_FACEBOOK = new Prefix("fb/");
    public static final Prefix PREFIX_TWITTER = new Prefix("tw/");
    public static final Prefix PREFIX_INSTAGRAM = new Prefix("in/");
    public static final Prefix PREFIX_GOOGLEPLUS = new Prefix("gp/");

}
```
###### \java\seedu\address\logic\parser\DeleteCommandParser.java
``` java
    public DeleteCommand parse(String args) throws ParseException {
        try {
            String[] arguments = args.trim().split(" ");
            if (arguments.length == 1) {
                Index index = ParserUtil.parseIndex(args);
                return new DeleteCommand(index);
            } else {
                Index[] indexes = new Index[arguments.length];
                for (int i = 0; i < indexes.length; i++) {
                    indexes[i] = ParserUtil.parseIndex(arguments[i]);
                }
                return new DeleteCommand(indexes);
            }
        } catch (IllegalValueException ive) {
            throw new ParseException(
                    String.format(MESSAGE_INVALID_COMMAND_FORMAT, DeleteCommand.MESSAGE_USAGE));
        }
    }

}
```
###### \java\seedu\address\model\ModelManager.java
``` java
    @Override
    public synchronized void deletePersons(ReadOnlyPerson[] targets) throws PersonNotFoundException {
        for (ReadOnlyPerson target : targets) {
            addressBook.removePerson(target);
        }
        updateFilteredPersonList(PREDICATE_SHOW_ALL_PERSONS);
        indicateAddressBookChanged();
    }
```
###### \java\seedu\address\model\schedule\exceptions\DuplicateScheduleException.java
``` java
/**
 * Signals that the operation will result in duplicate Schedule objects.
 */
public class DuplicateScheduleException extends DuplicateDataException {
    public DuplicateScheduleException() {
        super("Operation would result in duplicate schedules");
    }
}
```
###### \java\seedu\address\model\schedule\exceptions\NoSchedulesException.java
``` java
/**
 * Signals that the operation is unable to sort due to an empty list.
 */
public class NoSchedulesException extends Exception {}
```
###### \java\seedu\address\model\schedule\exceptions\ScheduleNotFoundException.java
``` java
/**
 * Signals that the operation is unable to find the specified person.
 */
public class ScheduleNotFoundException extends Exception {}
```
###### \java\seedu\address\model\schedule\UniqueScheduleList.java
``` java
/**
 * A list of schedules that enforces no nulls and uniqueness between its elements.
 *
 * Supports minimal set of list operations for the app's features.
 *
 * @see Schedule#equals(Object)
 */


public class UniqueScheduleList implements Iterable<Schedule> {

    private final ObservableList<Schedule> internalList = FXCollections.observableArrayList();
    // used by asObservableList()
    private final ObservableList<ReadOnlySchedule> mappedList = EasyBind.map(internalList, (schedule) -> schedule);

    /**
     * Creates a UniqueScheduleList using given Schedules.
     * Enforces no nulls.
     */
    public UniqueScheduleList(Set<Schedule> schedules) {
        requireAllNonNull(schedules);
        internalList.addAll(schedules);

        assert CollectionUtil.elementsAreUnique(internalList);
    }

    /**
     * Constructs empty ScheduleList.
     */
    public UniqueScheduleList() {}

    /**
     * Returns all schedules in this list as a Set.
     * This set is mutable and change-insulated against the internal list.
     */

    /**
     * Returns true if the list contains an equivalent Schedule as the given argument.
     */
    public boolean contains(ReadOnlySchedule toCheck) {
        requireNonNull(toCheck);
        return internalList.contains(toCheck);
    }

    /**
     * Returns a set representation of the schedule.
     */
    public Set<Schedule> toSet() {
        assert CollectionUtil.elementsAreUnique(internalList);
        /** Used LinkedHashSet to preserve insertion order **/
        return new LinkedHashSet<>(internalList);
    }

    /**
     * Ensures every schedule in the argument list exists in this object.
     */
    public void mergeFrom(UniqueScheduleList from) {
        final Set<Schedule> alreadyInside = this.toSet();
        from.internalList.stream()
                .filter(schedule -> !alreadyInside.contains(schedule))
                .forEach(internalList::add);

        assert CollectionUtil.elementsAreUnique(internalList);
    }

    /**
     * Adds a Schedule to the list.
     *
     * @throws seedu.address.model.schedule.exceptions.DuplicateScheduleException
     * if the Schedule to add is a duplicate of an existing Schedule in the list.
     */
    public void add(ReadOnlySchedule toAdd) throws DuplicateScheduleException {
        requireNonNull(toAdd);
        if (contains(toAdd)) {
            throw new DuplicateScheduleException();
        }

        internalList.add(new Schedule(toAdd));

        assert CollectionUtil.elementsAreUnique(internalList);
    }

    /**
     * Removes the equivalent schedule from the list.
     *
     * @throws ScheduleNotFoundException if no such schedule could be found in the list.
     */
    public boolean remove(ReadOnlySchedule toRemove) throws ScheduleNotFoundException {
        requireNonNull(toRemove);
        final boolean scheduleFoundAndDeleted = internalList.remove(toRemove);
        if (!scheduleFoundAndDeleted) {
            throw new ScheduleNotFoundException();
        }
        return scheduleFoundAndDeleted;
    }

    @Override
    public Iterator<Schedule> iterator() {
        assert CollectionUtil.elementsAreUnique(internalList);
        return internalList.iterator();
    }

    public void setSchedules(UniqueScheduleList replacement) {
        this.internalList.setAll(replacement.internalList);
    }

    public void setSchedules(List<? extends ReadOnlySchedule> schedules) throws DuplicateScheduleException {
        final UniqueScheduleList replacement = new UniqueScheduleList();
        for (final ReadOnlySchedule schedule: schedules) {
            replacement.add(new Schedule(schedule));
        }
        setSchedules(replacement);
    }

    /**
     * Returns the backing list as an unmodifiable {@code ObservableList}.
     */
    public ObservableList<ReadOnlySchedule> asObservableList() {
        assert CollectionUtil.elementsAreUnique(internalList);
        return FXCollections.unmodifiableObservableList(mappedList);
    }

    @Override
    public boolean equals(Object other) {
        assert CollectionUtil.elementsAreUnique(internalList);
        return other == this // short circuit if same object
                || (other instanceof seedu.address.model.schedule.UniqueScheduleList // instanceof handles nulls
                && this.internalList.equals(((seedu.address.model.schedule.UniqueScheduleList) other).internalList));
    }

    /**
     * Returns true if the element in this list is equal to the elements in {@code other}.
     * The elements do not have to be in the same order.
     */
    public boolean equalsOrderInsensitive(seedu.address.model.schedule.UniqueScheduleList other) {
        assert CollectionUtil.elementsAreUnique(internalList);
        assert CollectionUtil.elementsAreUnique(other.internalList);
        return this == other || new HashSet<>(this.internalList).equals(new HashSet<>(other.internalList));
    }

```
###### \java\seedu\address\storage\XmlAdaptedSchedule.java
``` java
/**
 * JAXB-friendly adapted version of the Schedule.
 */
public class XmlAdaptedSchedule {

    @XmlElement(required = true)
    private String scheduleName;
    @XmlElement(required = true)
    private String startDateTime;
    @XmlElement(required = true)
    private String endDateTime;
    @XmlElement(required = true)
    private String scheduleDuration;
    @XmlElement
    private String scheduleDetails;
    /**
     * Constructs an XmlAdaptedSchedule.
     * This is the no-arg constructor that is required by JAXB.
     */
    public XmlAdaptedSchedule() {}

    /**
     * Converts a given Schedule into this class for JAXB use.
     *
     * @param source future changes to this will not affect the created
     */
    public XmlAdaptedSchedule(ReadOnlySchedule source) {
        scheduleName = source.getName().toString();
        startDateTime = source.getStartDateTime().toString();
        endDateTime = source.getEndDateTime().toString();
        scheduleDuration = source.getScheduleDuration();
        scheduleDetails = source.getScheduleDetails();

    }

    /**
     * Converts this jaxb-friendly adapted schedule object into the model's Schedule object.
     *
     * @throws IllegalValueException if there were any data constraints violated in the adapted person
     */
    public Schedule toModelType() throws IllegalValueException {
        final ScheduleName scheduleName = new ScheduleName(this.scheduleName);
        final ScheduleDate startDateTime = new ScheduleDate(this.startDateTime);
        final ScheduleDate endDateTime = new ScheduleDate(this.endDateTime);
        final String scheduleDuration = this.scheduleDuration;
        final String scheduleDetails = this.scheduleDetails;
        return new Schedule(scheduleName, startDateTime, endDateTime, scheduleDuration, scheduleDetails);
    }

}
```
###### \java\seedu\address\ui\GroupCard.java
``` java
/**
 * An UI component that displays information of a {@code Person}.
 */
public class GroupCard extends UiPart<Region> {

    private static final String FXML = "GroupListCard.fxml";

    /**
     * Note: Certain keywords such as "location" and "resources" are reserved keywords in JavaFX.
     * As a consequence, UI elements' variable names cannot be set to such keywords
     * or an exception will be thrown by JavaFX during runtime.
     *
     * @see <a href="https://github.com/se-edu/addressbook-level4/issues/336">The issue on AddressBook level 4</a>
     */

    public final ReadOnlyGroup group;

    @FXML
    private Label groupName;
    @FXML
    private Label groupId;

    public GroupCard(ReadOnlyGroup group, int displayedIndex) {
        super(FXML);
        this.group = group;
        groupId.setText(displayedIndex + ". ");
        bindListeners(group);
    }

    /**
     * Binds the individual UI elements to observe their respective {@code Person} properties
     * so that they will be notified of any changes.
     */
    private void bindListeners(ReadOnlyGroup group) {
        groupName.textProperty().bind(Bindings.convert(group.nameProperty()));
    }

    @Override
    public boolean equals(Object other) {
        // short circuit if same object
        if (other == this) {
            return true;
        }

        // instanceof handles nulls
        if (!(other instanceof GroupCard)) {
            return false;
        }

        // state check
        GroupCard card = (GroupCard) other;
        return groupId.getText().equals(card.groupId.getText())
                && group.equals(card.group);
    }
}
```
###### \java\seedu\address\ui\GroupListPanel.java
``` java
/**
 * Panel containing the list of persons.
 */
public class GroupListPanel extends UiPart<Region> {
    private static final String FXML = "GroupListPanel.fxml";
    private final Logger logger = LogsCenter.getLogger(GroupListPanel.class);

    @FXML
    private ListView<GroupCard> groupListView;

    public GroupListPanel(ObservableList<ReadOnlyGroup> groupList) {
        super(FXML);
        setConnections(groupList);
        registerAsAnEventHandler(this);
    }

    private void setConnections(ObservableList<ReadOnlyGroup> groupList) {
        ObservableList<GroupCard> mappedList = EasyBind.map(
                groupList, (group) -> new GroupCard(group, groupList.indexOf(group) + 1));
        groupListView.setItems(mappedList);
        groupListView.setCellFactory(listView -> new GroupListViewCell());
        setEventHandlerForGroupSelectionChangeEvent();
    }

    private void setEventHandlerForGroupSelectionChangeEvent() {
        groupListView.getSelectionModel().selectedItemProperty()
                .addListener((observable, oldValue, newValue) -> {
                    if (newValue != null) {
                        logger.fine("Selection in group list panel changed to : '" + newValue + "'");
                        raise(new GroupPanelSelectionChangedEvent(newValue));
                    }
                });
    }

    /**
     * Scrolls to the {@code GroupCard} at the {@code index} and selects it.
     */
    private void scrollTo(int index) {
        Platform.runLater(() -> {
            groupListView.scrollTo(index);
            groupListView.getSelectionModel().clearAndSelect(index);
        });
    }

    @Subscribe
    private void handleJumpToGroupListRequestEvent(JumpToGroupListRequestEvent event) {
        logger.info(LogsCenter.getEventHandlingLogMessage(event));
        scrollTo(event.targetIndex);
        /** To ensure that group at index 1 can be selected even when previous selection was its index */
        groupListView.getSelectionModel().clearSelection();
    }

    /**
     * Custom {@code ListCell} that displays the graphics of a {@code GroupCard}.
     */
    class GroupListViewCell extends ListCell<GroupCard> {

        @Override
        protected void updateItem(GroupCard group, boolean empty) {
            super.updateItem(group, empty);

            if (empty || group == null) {
                setGraphic(null);
                setText(null);
            } else {
                setGraphic(group.getRoot());
            }
        }
    }

}
```
###### \java\seedu\address\ui\MainContactPanel.java
``` java
/**
 * The Browser Panel of the App.
 */
public class MainContactPanel extends UiPart<Region> {

    public static final String DEFAULT_PAGE = "default.html";

    private static final String FXML = "MainContactPanel.fxml";

    private final Logger logger = LogsCenter.getLogger(this.getClass());

    @FXML
    private Circle contactImageCircle;

    @FXML
    private BorderPane socialIcon1Placeholder;

    @FXML
    private BorderPane socialIcon2Placeholder;

    @FXML
    private BorderPane socialIcon3Placeholder;

    @FXML
    private BorderPane socialIcon4Placeholder;

    @FXML
    private BorderPane contactImagePlaceholder;

    @FXML
    private VBox contactDetailsVBox;

    @FXML
    private StackPane schedulePlaceholder;

    //This is needed for setting the click listener in setIcons(), if not the
    //circles won't be able to pass a parameter for the method it calls inside
    //its listener.
    private ReadOnlyPerson currentPerson;

    private ParallelTransition pt;

    public MainContactPanel() {
        super(FXML);
        // To prevent triggering events for typing inside the loaded Web page.
        getRoot().setOnKeyPressed(Event::consume);
        //Setup needed JFX nodes which will be updated upon selecting persons
        setupContactImageCircle();
        setupContactDetailsVBox();
        setupScheduleListViewPlaceholder();
        registerAsAnEventHandler(this);
    }

```
###### \java\seedu\address\ui\MainContactPanel.java
``` java

    private void setupContactImageCircle() {
        contactImageCircle = new Circle(250, 250, 90);
        contactImageCircle.setStroke(Color.valueOf("#3fc380"));
        contactImageCircle.setStrokeWidth(5);
        contactImageCircle.radiusProperty().bind(Bindings.min(
                contactImagePlaceholder.widthProperty().divide(3),
                contactImagePlaceholder.heightProperty().divide(3))
        );
        contactImagePlaceholder.setCenter(contactImageCircle);
        contactImageCircle.setVisible(false);
    }

    private void setupContactDetailsVBox() {
        contactDetailsVBox.setSpacing(0);
        contactDetailsVBox.getChildren().addAll(
                new Label(""),
                new Label(""),
                new Label(""),
                new Label("")
        );
        contactDetailsVBox.setStyle("-fx-alignment: center-left; -fx-padding: 0 0 0 10");
    }

    private void setIcons() {
        BorderPane[] socialIconPlaceholders = {
            socialIcon1Placeholder,
            socialIcon2Placeholder,
            socialIcon3Placeholder,
            socialIcon4Placeholder
        };
        String[] imgUrls = {
            "images/facebook.png",
            "images/twitter.png",
            "images/instagram.png",
            "images/googleplus.png"
        };

        for (int i = 0; i < 4; i++) {
            Circle cir = new Circle(250, 250, 30);
            cir.setStroke(Color.valueOf("#3fc380"));
            cir.setStrokeWidth(5);
            cir.radiusProperty().bind(Bindings.min(
                    socialIconPlaceholders[i].widthProperty().divide(3),
                    socialIconPlaceholders[i].heightProperty().divide(3))
            );
            /*
            //V2.0 FEATURE - SOCIAL MEDIA, NOT WORKING YET
            //Set up mouse click listeners to run method to open social pages
            cir.setOnMouseClicked(new EventHandler<MouseEvent>() {
                @Override
                public void handle(MouseEvent event) {
                    openSocialIconPage(currentPerson);
                }
            });
            */
            cir.setFill(new ImagePattern(new Image(imgUrls[i])));
            socialIconPlaceholders[i].setCenter(cir);
            easeIn(cir);
        }
    }

    /**
     * NOTE: V2.0 FEATURE - SOCIAL MEDIA, NO WORKING YET
     * Loads the social page in a new window.
     * There is no controller file for the social media window fxml
     * as it is essentially only a WebView.
     */
    private void openSocialIconPage(ReadOnlyPerson person) {
        try {
            //Load the component
            FXMLLoader fxmlLoader = new FXMLLoader();
            fxmlLoader.setLocation(getClass().getResource("/view/SocialMediaPageWindow.fxml"));
            AnchorPane parent = fxmlLoader.load();
            //Get the webview from the loaded component then put URL
            WebView socialPageView = (WebView) parent.getChildren().get(0);
            socialPageView.getEngine().load(person.getName().fullName);
            //Create the scene and stage
            Scene scene = new Scene(parent);
            Stage stage = new Stage();
            //Setup window size based on the user's screen size
            Rectangle2D screenBounds = Screen.getPrimary().getVisualBounds();
            stage.setWidth(screenBounds.getWidth() / 1.4);
            stage.setHeight(screenBounds.getHeight() / 1.2);
            //Set title and show the scene
            stage.setTitle("Social Media Window");
            stage.setScene(scene);
            stage.show();
        } catch (IOException e) {
            Logger logger = Logger.getLogger(getClass().getName());
            logger.log(Level.SEVERE, "Failed to create new window for social media page.", e);
        }
    }

    private void setupScheduleListViewPlaceholder() {
        schedulePlaceholder.setVisible(false);
    }

    @Subscribe
    private void handlePersonPanelSelectionChangedEvent(PersonPanelSelectionChangedEvent event)
            throws MalformedURLException {
        logger.info(LogsCenter.getEventHandlingLogMessage(event));
        setContactImage(event.getNewSelection().person);
        setContactDetails(event.getNewSelection().person);
        setIcons();
        setSchedule(event.getNewSelection().person);
    }

    private void setContactDetails(ReadOnlyPerson person) {
        //Set up name label separately as it has no icons
        contactDetailsVBox.setSpacing(0);
        contactDetailsVBox.getChildren().addAll();

        Label name = (Label) contactDetailsVBox.getChildren().get(0);
        name.setText("" + person.getName());
        name.setStyle("-fx-font-size: 60;");
        name.setWrapText(true);
        easeIn(name);

        //Set values of other labels
        Label phone = (Label) contactDetailsVBox.getChildren().get(1);
        phone.setText("" + person.getPhone());
        Label email = (Label) contactDetailsVBox.getChildren().get(2);
        email.setText("" + person.getEmail());
        Label address = (Label) contactDetailsVBox.getChildren().get(3);
        address.setText("" + person.getAddress());

        //Add images to these labels
        Label[] labels = {phone, email, address};
        String[] iconUrls = {"images/phone.png", "images/mail.png", "images/homeBlack.png"};
        for (int i = 0; i < labels.length; i++) {
            ImageView icon = new ImageView(iconUrls[i]);
            icon.setImage(new Image(iconUrls[i]));
            icon.setPreserveRatio(true);
            icon.setFitWidth(20);
            labels[i].setGraphic(icon);
            labels[i].setWrapText(true);
            labels[i].setStyle("-fx-font-size: 17");
            easeIn(labels[i]);
        }
        currentPerson = person;
    }

    private void setSchedule(ReadOnlyPerson person) {
        schedulePlaceholder.setVisible(true);
        ScheduleListPanel scheduleList = new ScheduleListPanel(person.scheduleProperty().get().asObservableList());
        schedulePlaceholder.getChildren().add(scheduleList.getRoot());
        //scheduleListView.setStyle("-fx-alignment: center-left; -fx-padding: 0 0 0 10;");
        //easeIn(schedulePlaceholder);
        easeIn(schedulePlaceholder);
        currentPerson = person;
    }

    /**
     * Animates any node passed into this method with an ease-in
     */
    private void easeIn(Node node) {
        FadeTransition ft = new FadeTransition(Duration.millis(500), node);
        ft.setFromValue(0);
        ft.setToValue(1);
        TranslateTransition tt = new TranslateTransition();
        tt.setNode(node);
        tt.setFromY(20);
        tt.setToY(0);
        tt.setDuration(Duration.millis(500));
        tt.setInterpolator(Interpolator.EASE_IN);
        ParallelTransition pt = new ParallelTransition();
        pt.getChildren().addAll(ft, tt);
        pt.play();
    }

    public StackPane getSchedulePlaceholder() {
        return schedulePlaceholder;
    }
}
```
###### \java\seedu\address\ui\ScheduleCard.java
``` java
/**
 * An UI component that displays information of a {@code Schedule}.
 */
public class ScheduleCard extends UiPart<Region> {

    private static final String FXML = "ScheduleListCard.fxml";

    /**
     * Note: Certain keywords such as "location" and "resources" are reserved keywords in JavaFX.
     * As a consequence, UI elements' variable names cannot be set to such keywords
     * or an exception will be thrown by JavaFX during runtime.
     *
     * @see <a href="https://github.com/se-edu/addressbook-level4/issues/336">The issue on AddressBook level 4</a>
     */

    public final ReadOnlySchedule schedule;

    @FXML
    private Label scheduleName;
    @FXML
    private Label scheduleId;
    @FXML
    private Label scheduleDuration;
    @FXML
    private Label scheduleDetails;

    public ScheduleCard(ReadOnlySchedule schedule, int displayedIndex) {
        super(FXML);
        this.schedule = schedule;
        scheduleId.setText(displayedIndex + ". ");
        bindListeners(schedule);
    }

    /**
     * Binds the individual UI elements to observe their respective {@code Schedule} properties
     * so that they will be notified of any changes.
     */
    private void bindListeners(ReadOnlySchedule schedule) {
        scheduleName.textProperty().bind(Bindings.convert(schedule.nameProperty()));
        scheduleDuration.textProperty().bind(Bindings.convert(schedule.scheduleDurationProperty()));
        scheduleDetails.textProperty().bind(Bindings.convert(schedule.scheduleDetailsProperty()));
    }

    @Override
    public boolean equals(Object other) {
        // short circuit if same object
        if (other == this) {
            return true;
        }

        // instanceof handles nulls
        if (!(other instanceof ScheduleCard)) {
            return false;
        }

        // state check
        ScheduleCard card = (ScheduleCard) other;
        return scheduleId.getText().equals(card.scheduleId.getText())
                && schedule.equals(card.schedule);
    }
}
```
###### \java\seedu\address\ui\ScheduleListPanel.java
``` java
/**
 * Panel containing the list of schedules.
 */
public class ScheduleListPanel extends UiPart<Region> {
    private static final String FXML = "ScheduleListPanel.fxml";
    private final Logger logger = LogsCenter.getLogger(ScheduleListPanel.class);

    @FXML
    private ListView<ScheduleCard> scheduleListView;

    public ScheduleListPanel() {
        super(FXML);
    }

    public ScheduleListPanel(ObservableList<ReadOnlySchedule> scheduleList) {
        super(FXML);
        setConnections(scheduleList);
        registerAsAnEventHandler(this);
    }

    private void setConnections(ObservableList<ReadOnlySchedule> scheduleList) {
        ObservableList<ScheduleCard> mappedList = EasyBind.map(
                scheduleList, (schedule) -> new ScheduleCard(schedule, scheduleList.indexOf(schedule) + 1));
        scheduleListView.setItems(mappedList);
        scheduleListView.setCellFactory(listView -> new ScheduleListViewCell());
        setEventHandlerForSelectionChangeEvent();
    }

    private void setEventHandlerForSelectionChangeEvent() {
        scheduleListView.getSelectionModel().selectedItemProperty()
                .addListener((observable, oldValue, newValue) -> {
                    if (newValue != null) {
                        logger.fine("Selection in schedule list panel changed to : '" + newValue + "'");
                        raise(new SchedulePanelSelectionChangedEvent(newValue));
                    }
                });
    }


    /**
     * Custom {@code ListCell} that displays the graphics of a {@code ScheduleCard}.
     */
    class ScheduleListViewCell extends ListCell<ScheduleCard> {

        @Override
        protected void updateItem(ScheduleCard schedule, boolean empty) {
            super.updateItem(schedule, empty);

            if (empty || schedule == null) {
                setGraphic(null);
                setText(null);
            } else {
                setGraphic(schedule.getRoot());
            }
        }
    }

}
```
###### \resources\view\GroupListCard.fxml
``` fxml
<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.layout.ColumnConstraints?>
<?import javafx.scene.layout.FlowPane?>
<?import javafx.scene.layout.GridPane?>
<?import javafx.scene.layout.HBox?>
<?import javafx.scene.layout.Region?>
<?import javafx.scene.layout.RowConstraints?>
<?import javafx.scene.layout.VBox?>

<HBox id="groupCardPane" fx:id="groupCardPane" xmlns="http://javafx.com/javafx/8.0.111" xmlns:fx="http://javafx.com/fxml/1">
  <GridPane HBox.hgrow="ALWAYS">
    <columnConstraints>
      <ColumnConstraints hgrow="SOMETIMES" minWidth="10" prefWidth="150" />
    </columnConstraints>
    <VBox alignment="CENTER_LEFT" GridPane.columnIndex="0">
      <padding>
        <Insets bottom="5" left="15" right="5" top="5" />
      </padding>
      <HBox alignment="CENTER_LEFT" spacing="5">
        <Label fx:id="groupId" styleClass="cell_big_label">
          <minWidth>
            <!-- Ensures that the label text is never truncated -->
            <Region fx:constant="USE_PREF_SIZE" />
          </minWidth>
        </Label>
        <Label fx:id="groupName" styleClass="cell_big_label" text="\$first" />
      </HBox>
      <FlowPane fx:id="groupTags" />
    </VBox>
      <rowConstraints>
         <RowConstraints />
      </rowConstraints>
  </GridPane>
</HBox>
```
###### \resources\view\GroupListPanel.fxml
``` fxml
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.ListView?>
<?import javafx.scene.layout.VBox?>

<VBox xmlns="http://javafx.com/javafx/8.0.111" xmlns:fx="http://javafx.com/fxml/1" style="-fx-border-width: 1">
   <Label text="Groups" styleClass="panelTitlePadding"/>
  <ListView fx:id="groupListView" VBox.vgrow="ALWAYS" />
</VBox>
```
###### \resources\view\MainContactPanel.fxml
``` fxml
<?import javafx.scene.layout.BorderPane?>
<?import javafx.scene.layout.ColumnConstraints?>
<?import javafx.scene.layout.GridPane?>
<?import javafx.scene.layout.RowConstraints?>
<?import javafx.scene.layout.StackPane?>
<?import javafx.scene.layout.VBox?>

<StackPane xmlns="http://javafx.com/javafx/8.0.111" xmlns:fx="http://javafx.com/fxml/1">
   <GridPane fx:id="mainDetailsPane" gridLinesVisible="false">
     <columnConstraints>
       <ColumnConstraints hgrow="SOMETIMES" minWidth="10.0" prefWidth="100.0" />
       <ColumnConstraints hgrow="SOMETIMES" minWidth="10.0" prefWidth="100.0" />
     </columnConstraints>
     <rowConstraints>
       <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
       <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
     </rowConstraints>
      <children>
        <BorderPane fx:id="contactImagePlaceholder" prefHeight="150.0" prefWidth="200.0" styleClass="centerImagesInStackPane" />
         <GridPane fx:id="socialIconsPane" gridLinesVisible="false" GridPane.rowIndex="1">
           <columnConstraints>
             <ColumnConstraints hgrow="SOMETIMES" minWidth="10.0" prefWidth="100.0" />
             <ColumnConstraints hgrow="SOMETIMES" minWidth="10.0" prefWidth="100.0" />
           </columnConstraints>
           <rowConstraints>
             <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
             <RowConstraints minHeight="10.0" prefHeight="30.0" vgrow="SOMETIMES" />
           </rowConstraints>
            <children>
              <BorderPane fx:id="socialIcon1Placeholder" prefHeight="150.0" prefWidth="200.0" styleClass="centerImagesInStackPane" GridPane.halignment="CENTER" />
              <BorderPane fx:id="socialIcon2Placeholder" prefHeight="150.0" prefWidth="200.0" styleClass="centerImagesInStackPane" GridPane.columnIndex="1" />
              <BorderPane fx:id="socialIcon3Placeholder" prefHeight="150.0" prefWidth="200.0" styleClass="centerImagesInStackPane" GridPane.rowIndex="1" />
              <BorderPane fx:id="socialIcon4Placeholder" prefHeight="150.0" prefWidth="200.0" styleClass="centerImagesInStackPane" GridPane.columnIndex="1" GridPane.rowIndex="1" />
            </children>
         </GridPane>
         <VBox fx:id="contactDetailsVBox" prefHeight="200.0" prefWidth="100.0" GridPane.columnIndex="1" />
              <StackPane fx:id="schedulePlaceholder" styleClass="panelTitlePadding" GridPane.columnIndex="1" GridPane.rowIndex="1" />
      </children>
   </GridPane>
</StackPane>
```
###### \resources\view\MainWindow.fxml
``` fxml
    <VBox fx:id="groupList" minWidth="340" prefWidth="340" SplitPane.resizableWithParent="false">
         <padding>
            <Insets bottom="10" left="10" right="10" top="10" />
         </padding>
            <StackPane fx:id="groupListPanelPlaceholder" VBox.vgrow="ALWAYS" styleClass="individualBorderInSplitPane"/>
     </VBox>

   <StackPane fx:id="mainContactPanel" VBox.vgrow="ALWAYS">
       <padding>
           <Insets bottom="10" left="10" right="10" top="10" />
       </padding>
    <StackPane fx:id="browserPlaceholder" minWidth="340" prefWidth="340" styleClass="individualBorderInSplitPane">
      <padding>
        <Insets bottom="10" left="10" right="10" top="10" />
      </padding>
    </StackPane>
   </StackPane>

  </SplitPane>

  <StackPane fx:id="statusbarPlaceholder" VBox.vgrow="NEVER" />
</VBox>
```
###### \resources\view\ScheduleListCard.fxml
``` fxml
<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.layout.ColumnConstraints?>
<?import javafx.scene.layout.FlowPane?>
<?import javafx.scene.layout.GridPane?>
<?import javafx.scene.layout.HBox?>
<?import javafx.scene.layout.Region?>
<?import javafx.scene.layout.RowConstraints?>
<?import javafx.scene.layout.VBox?>

<HBox id="scheduleCardPane" fx:id="scheduleCardPane" xmlns="http://javafx.com/javafx/8.0.111" xmlns:fx="http://javafx.com/fxml/1">
  <GridPane HBox.hgrow="ALWAYS">
    <columnConstraints>
      <ColumnConstraints hgrow="SOMETIMES" minWidth="10" prefWidth="150" />
    </columnConstraints>
    <VBox alignment="CENTER_LEFT" GridPane.columnIndex="0">
      <padding>
        <Insets bottom="5" left="15" right="5" top="5" />
      </padding>
      <HBox alignment="CENTER_LEFT" spacing="5">
        <Label fx:id="scheduleId" styleClass="cell_big_label">
          <minWidth>
            <!-- Ensures that the label text is never truncated -->
            <Region fx:constant="USE_PREF_SIZE" />
          </minWidth>
        </Label>
        <Label fx:id="scheduleName" styleClass="cell_big_label" text="\$first" />
      </HBox>
      <FlowPane fx:id="scheduleTags" />
      <Label fx:id="scheduleDuration" styleClass="cell_small_label" text="\$duration" />
      <Label fx:id="scheduleDetails" styleClass="cell_small_label" text="\$details" />
    </VBox>
      <rowConstraints>
         <RowConstraints />
      </rowConstraints>
  </GridPane>
</HBox>
```
###### \resources\view\ScheduleListPanel.fxml
``` fxml
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.ListView?>
<?import javafx.scene.layout.VBox?>

<VBox style="-fx-border-width: 1; -fx-border-color:black; -fx-border-radius: 10; -fx-background-radius: 10;
 -fx-border-color: white;" xmlns="http://javafx.com/javafx/8.0.111" xmlns:fx="http://javafx.com/fxml/1">
   <Label fx:id="scheduleLabel" prefWidth="${scheduleLabel.parent.width}" styleClass="panelTitlePadding" text="Schedule" />
  <ListView fx:id="scheduleListView" VBox.vgrow="ALWAYS" />
</VBox>
```
###### \resources\view\SocialMediaPageWindow.fxml
``` fxml
<?import javafx.scene.layout.AnchorPane?>
<?import javafx.scene.web.WebView?>


<AnchorPane prefHeight="400.0" prefWidth="600.0" xmlns="http://javafx.com/javafx/8.0.111" xmlns:fx="http://javafx.com/fxml/1">
   <children>
      <WebView fx:id="socialMediaView" layoutX="94.0" layoutY="63.0" prefHeight="200.0" prefWidth="200.0" AnchorPane.bottomAnchor="0.0" AnchorPane.leftAnchor="0.0" AnchorPane.rightAnchor="0.0" AnchorPane.topAnchor="0.0" />
   </children>
</AnchorPane>
```
###### \resources\view\UITheme.css
``` css
.background {
    -fx-background-color: derive(#fafafa, 20%);
    background-color: #fafafa; /* Used in the default.html file */
}

.label {
    -fx-font-size: 11pt;
    -fx-font-family: "Segoe UI Semibold";
    -fx-text-fill: #000000;
    -fx-opacity: 0.9;
}

.label-bright {
    -fx-font-size: 11pt;
    -fx-font-family: "Segoe UI Semibold";
    -fx-text-fill: black;
    -fx-opacity: 1;
}

.label-header {
    -fx-font-size: 32pt;
    -fx-font-family: "Segoe UI Light";
    -fx-text-fill: black;
    -fx-opacity: 1;
}

.text-field {
    -fx-font-size: 12pt;
    -fx-font-family: "Segoe UI Semibold";
}

.tab-pane {
    -fx-padding: 0 0 0 1;
}

.tab-pane .tab-header-area {
    -fx-padding: 0 0 0 0;
    -fx-min-height: 0;
    -fx-max-height: 0;
}

.table-view {
    -fx-base: #1d1d1d;
    -fx-control-inner-background: #1d1d1d;
    -fx-background-color: #1d1d1d;
    -fx-table-cell-border-color: transparent;
    -fx-table-header-border-color: transparent;
    -fx-padding: 5;
}

.table-view .column-header-background {
    -fx-background-color: transparent;
}

.table-view .column-header, .table-view .filler {
    -fx-size: 35;
    -fx-border-width: 0 0 1 0;
    -fx-background-color: transparent;
    -fx-border-color:
        transparent
        transparent
        derive(-fx-base, 80%)
        transparent;
    -fx-border-insets: 0 10 1 0;
}

.table-view .column-header .label {
    -fx-font-size: 20pt;
    -fx-font-family: "Segoe UI Light";
    -fx-text-fill: black;
    -fx-alignment: center-left;
    -fx-opacity: 1;
}

.table-view:focused .table-row-cell:filled:focused:selected {
    -fx-background-color: -fx-focus-color;
}

.split-pane:horizontal .split-pane-divider {
    -fx-background-color: derive(#ffffff, 20%);
    -fx-border-color: transparent transparent transparent transparent;
}

.split-pane {
    -fx-border-radius: 1;
    -fx-border-width: 1;
    -fx-background-color: derive(#fafafa, 20%);
}

.list-view {
    -fx-background-insets: 0;
    -fx-padding: 0;
}

.list-cell {
    -fx-label-padding: 0 0 0 0;
    -fx-graphic-text-gap : 0;
    -fx-padding: 0 0 0 0;
}

.list-cell:filled:even {
    -fx-background-color: derive(#ecf0f1, 0%);
}

.list-cell:filled:odd {
    -fx-background-color: derive(#ecf0f1, 50%);
}

.list-cell:filled:selected {
    -fx-background-color: derive(#3fc380, 20%);
}

.list-cell:filled:selected #cardPane {
    -fx-border-color: #3e7b91;
    -fx-border-width: 0;
}

.list-cell .label {
    -fx-text-fill: black;
}

.cell_big_label {
    -fx-font-family: "Segoe UI Semibold";
    -fx-font-size: 16px;
    -fx-text-fill: #010504;
}

.cell_small_label {
    -fx-font-family: "Segoe UI";
    -fx-font-size: 13px;
    -fx-text-fill: #010504;
}

.anchor-pane {
     -fx-background-color: derive(#fafafa, 20%);
}

.pane-with-border {
     -fx-background-color: derive(#fafafa, 20%);
     -fx-border-color: derive(#fafafa, 10%);
     -fx-border-top-width: 1px;
}

.status-bar {
    -fx-background-color: derive(#1d1d1d, 20%);
    -fx-text-fill: black;
}

.result-display {
    -fx-background-color: transparent;
    -fx-font-family: "Segoe UI Light";
    -fx-font-size: 13pt;
    -fx-text-fill: black;
}

.result-display .label {
    -fx-text-fill: black !important;
}

.status-bar .label {
    -fx-font-family: "Segoe UI Light";
    -fx-text-fill: black;
}

.status-bar-with-border {
    -fx-background-color: derive(#1d1d1d, 30%);
    -fx-border-color: derive(#1d1d1d, 25%);
    -fx-border-width: 1px;
}

.status-bar-with-border .label {
    -fx-text-fill: black;
}

.grid-pane {
    -fx-background-color: derive(#ffffff, 30%);
    -fx-border-color: derive(#ffffff, 30%);
    -fx-border-width: 1px;
}

.grid-pane .anchor-pane {
    -fx-background-color: derive(#ffffff, 30%);
}

.context-menu {
    -fx-background-color: derive(#3fc380, 0%);
}

.context-menu .label {
    -fx-text-fill: black;
}

.menu-bar {
    -fx-background-color: derive(#3fc380, 0%);
}

.menu-bar .label {
    -fx-font-size: 14pt;
    -fx-font-family: "Segoe UI Light";
    -fx-text-fill: black;
    -fx-opacity: 0.9;
}

.menu .left-container {
    -fx-background-color: black;
}

.menu {
    -fx-accent: derive(#3fc380, 50%);
}

.menu-item {
    -fx-accent: derive(#3fc380, 50%);
}

/*
 * Metro style Push Button
 * Author: Pedro Duque Vieira
 * http://pixelduke.wordpress.com/2012/10/23/jmetro-windows-8-controls-on-java/
 */
.button {
    -fx-padding: 5 22 5 22;
    -fx-border-color: #e2e2e2;
    -fx-border-width: 2;
    -fx-background-radius: 0;
    -fx-background-color: #1d1d1d;
    -fx-font-family: "Segoe UI", Helvetica, Arial, sans-serif;
    -fx-font-size: 11pt;
    -fx-text-fill: #d8d8d8;
    -fx-background-insets: 0 0 0 0, 0, 1, 2;
}

.button:hover {
    -fx-background-color: #3a3a3a;
}

.button:pressed, .button:default:hover:pressed {
  -fx-background-color: white;
  -fx-text-fill: #1d1d1d;
}

.button:focused {
    -fx-border-color: white, white;
    -fx-border-width: 1, 1;
    -fx-border-style: solid, segments(1, 1);
    -fx-border-radius: 0, 0;
    -fx-border-insets: 1 1 1 1, 0;
}

.button:disabled, .button:default:disabled {
    -fx-opacity: 0.4;
    -fx-background-color: #1d1d1d;
    -fx-text-fill: black;
}

.button:default {
    -fx-background-color: -fx-focus-color;
    -fx-text-fill: #ffffff;
}

.button:default:hover {
    -fx-background-color: derive(-fx-focus-color, 30%);
}

.dialog-pane {
    -fx-background-color: #1d1d1d;
}

.dialog-pane > *.button-bar > *.container {
    -fx-background-color: #1d1d1d;
}

.dialog-pane > *.label.content {
    -fx-font-size: 14px;
    -fx-font-weight: bold;
    -fx-text-fill: black;
}

.dialog-pane:header *.header-panel {
    -fx-background-color: derive(#1d1d1d, 25%);
}

.dialog-pane:header *.header-panel *.label {
    -fx-font-size: 18px;
    -fx-font-style: italic;
    -fx-fill: white;
    -fx-text-fill: black;
}

.scroll-bar {
    -fx-background-color: #ffffff;
}

.scroll-bar .thumb {
    -fx-background-color: #3fc380;
    -fx-background-insets: 3;
}

.scroll-bar .increment-button, .scroll-bar .decrement-button {
    -fx-background-color: transparent;
    -fx-padding: 0 0 0 0;
}

.scroll-bar .increment-arrow, .scroll-bar .decrement-arrow {
    -fx-shape: " ";
}

.scroll-bar:vertical .increment-arrow, .scroll-bar:vertical .decrement-arrow {
    -fx-padding: 1 8 1 8;
}

.scroll-bar:horizontal .increment-arrow, .scroll-bar:horizontal .decrement-arrow {
    -fx-padding: 8 1 8 1;
}

#cardPane {
    -fx-background-color: transparent;
    -fx-border-width: 0;
}

#commandTypeLabel {
    -fx-font-size: 11px;
    -fx-text-fill: #F70D1A;
}

#commandTextField {
    -fx-background-color: transparent transparent transparent transparent;
    -fx-background-insets: 0;
    -fx-border-color: #3fc380 #3fc380 #3fc380 #3fc380;
    -fx-border-insets: 0;
    -fx-border-width: 2;
    -fx-font-family: "Segoe UI Light";
    -fx-font-size: 13pt;
    -fx-text-fill: black;
}

#filterField, #personListPanel, #personWebpage {
    -fx-effect: innershadow(gaussian, black, 10, 0, 0, 0);
}

#resultDisplay .content {
    -fx-background-color: transparent, #ffffff, transparent, #ffffff;
    -fx-background-radius: 0;
    -fx-border-color: #3fc380;
    -fx-border-width: 2;
    -fx-border-radius: 20;
}

#tags {
    -fx-hgap: 7;
    -fx-vgap: 3;
}

#tags .label{
    -fx-text-fill: black;
    -fx-background-color: #3e7b91;
    -fx-padding: 1 3 1 3;
    -fx-border-radius: 2;
    -fx-background-radius: 5;
    -fx-font-size: 11;
}

.individualBorderInSplitPane {
    -fx-border-width: 1;
    -fx-border-color: black;
    -fx-border-radius: 5;
}

.panelTitlePadding{
    -fx-padding: 10 10 10 10 ;
}

.centerImagesInStackPane{
    -fx-background-position: center center;
    -fx-background-repeat: no-repeat;
    -fx-background-size: contain;
    -fx-background-radius: 4;
}
```
