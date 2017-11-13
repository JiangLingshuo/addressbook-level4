# Procrastinatus
###### \java\guitests\guihandles\GroupListPanelHandle.java
``` java
/**
 * Provides a handle for {@code GroupListPanel} containing the list of {@code GroupCard}.
 */
public class GroupListPanelHandle extends NodeHandle<ListView<GroupCard>> {
    public static final String GROUP_LIST_VIEW_ID = "#groupListView";

    private Optional<GroupCard> lastRememberedSelectedGroupCard;

    public GroupListPanelHandle(ListView<GroupCard> groupListPanelNode) {
        super(groupListPanelNode);
    }

    /**
     * Returns a handle to the selected {@code GroupCardHandle}.
     * A maximum of 1 item can be selected at any time.
     * @throws AssertionError if no card is selected, or more than 1 card is selected.
     */
    public GroupCardHandle getHandleToSelectedCard() {
        List<GroupCard> groupList = getRootNode().getSelectionModel().getSelectedItems();
        if (groupList.size() != 1) {
            throw new AssertionError("Group list size expected 1.");
        }

        return new GroupCardHandle(groupList.get(0).getRoot());
    }

    /**
     * Returns the index of the selected card.
     */
    public int getSelectedCardIndex() {
        return getRootNode().getSelectionModel().getSelectedIndex();
    }

    /**
     * Returns true if a card is currently selected.
     */
    public boolean isAnyCardSelected() {
        List<GroupCard> selectedCardsList = getRootNode().getSelectionModel().getSelectedItems();

        if (selectedCardsList.size() > 1) {
            throw new AssertionError("Card list size expected 0 or 1.");
        }

        return !selectedCardsList.isEmpty();
    }

    /**
     * Navigates the listview to display and select the group.
     */
    public void navigateToCard(ReadOnlyGroup group) {
        List<GroupCard> cards = getRootNode().getItems();
        Optional<GroupCard> matchingCard = cards.stream().filter(card -> card.group.equals(group)).findFirst();

        if (!matchingCard.isPresent()) {
            throw new IllegalArgumentException("Group does not exist.");
        }

        guiRobot.interact(() -> {
            getRootNode().scrollTo(matchingCard.get());
            getRootNode().getSelectionModel().select(matchingCard.get());
        });
        guiRobot.pauseForHuman();
    }

    /**
     * Returns the group card handle of a group associated with the {@code index} in the list.
     */
    public GroupCardHandle getGroupCardHandle(int index) {
        return getGroupCardHandle(getRootNode().getItems().get(index).group);
    }

    /**
     * Returns the {@code GroupCardHandle} of the specified {@code group} in the list.
     */
    public GroupCardHandle getGroupCardHandle(ReadOnlyGroup group) {
        Optional<GroupCardHandle> handle = getRootNode().getItems().stream()
                .filter(card -> card.group.equals(group))
                .map(card -> new GroupCardHandle(card.getRoot()))
                .findFirst();
        return handle.orElseThrow(() -> new IllegalArgumentException("Group does not exist."));
    }

    /**
     * Selects the {@code GroupCard} at {@code index} in the list.
     */
    public void select(int index) {
        getRootNode().getSelectionModel().select(index);
    }

    /**
     * Remembers the selected {@code GroupCard} in the list.
     */
    public void rememberSelectedGroupCard() {
        List<GroupCard> selectedItems = getRootNode().getSelectionModel().getSelectedItems();

        if (selectedItems.size() == 0) {
            lastRememberedSelectedGroupCard = Optional.empty();
        } else {
            lastRememberedSelectedGroupCard = Optional.of(selectedItems.get(0));
        }
    }

    /**
     * Returns true if the selected {@code GroupCard} is different from the value remembered by the most recent
     * {@code rememberSelectedGroupCard()} call.
     */
    public boolean isSelectedGroupCardChanged() {
        List<GroupCard> selectedItems = getRootNode().getSelectionModel().getSelectedItems();

        if (selectedItems.size() == 0) {
            return lastRememberedSelectedGroupCard.isPresent();
        } else {
            return !lastRememberedSelectedGroupCard.isPresent()
                    || !lastRememberedSelectedGroupCard.get().equals(selectedItems.get(0));
        }
    }

    /**
     * Returns the size of the list.
     */
    public int getListSize() {
        return getRootNode().getItems().size();
    }
}
```
###### \java\seedu\address\TestApp.java
``` java
        userPrefs.updateLastUsedGuiSetting(new GuiSettings(1000.0, 600.0, (int) x, (int) y));
```
###### \java\seedu\address\ui\GroupCardTest.java
``` java
public class GroupCardTest extends GuiUnitTest {

    @Test
    public void display() {
        Group group = new GroupBuilder().build();
        GroupCard groupCard = new GroupCard(group, 1);
        uiPartRule.setUiPart(groupCard);
        assertCardDisplay(groupCard, group, 1);

        Group group2 = new GroupBuilder().build();
        groupCard = new GroupCard(group2, 2);
        uiPartRule.setUiPart(groupCard);
        assertCardDisplay(groupCard, group2, 2);

        // changes made to Group reflects on card
        guiRobot.interact(() -> {
            group2.setGroupName(ALICE_GROUP.getName());
        });
        assertCardDisplay(groupCard, group2, 2);
    }

    @Test
    public void equals() {
        Group group = new GroupBuilder().build();
        GroupCard groupCard = new GroupCard(group, 0);

        // same group, same index -> returns true
        GroupCard copy = new GroupCard(group, 0);
        assertTrue(groupCard.equals(copy));

        // same object -> returns true
        assertTrue(groupCard.equals(groupCard));

        // null -> returns false
        assertFalse(groupCard == null);

        // different types -> returns false
        assertFalse(groupCard.equals(0));

        // different group, same index -> returns false
        Group differentGroup = new GroupBuilder().withName("differentName").build();
        assertFalse(groupCard.equals(new GroupCard(differentGroup, 0)));

        // same group, different index -> returns false
        assertFalse(groupCard.equals(new GroupCard(group, 1)));
    }

    /**
     * Asserts that {@code groupCard} displays the details of {@code expectedGroup} correctly and matches
     * {@code expectedId}.
     */
    private void assertCardDisplay(GroupCard groupCard, ReadOnlyGroup expectedGroup, int expectedId) {
        guiRobot.pauseForHuman();

        GroupCardHandle groupCardHandle = new GroupCardHandle(groupCard.getRoot());

        // verify id is displayed correctly
        assertEquals(Integer.toString(expectedId) + ". ", groupCardHandle.getId());

        // verify group details are displayed correctly
        assertCardDisplaysGroup(expectedGroup, groupCardHandle);
    }
}
```
###### \java\seedu\address\ui\GroupListPanelTest.java
``` java
public class GroupListPanelTest extends GuiUnitTest {
    private static final ObservableList<ReadOnlyGroup> TYPICAL_GROUPS =
            FXCollections.observableList(getTypicalGroups());

    private static final JumpToGroupListRequestEvent JUMP_TO_SECOND_EVENT =
            new JumpToGroupListRequestEvent(INDEX_SECOND_GROUP);

    private GroupListPanelHandle groupListPanelHandle;

    @Before
    public void setUp() {
        GroupListPanel groupListPanel = new GroupListPanel(TYPICAL_GROUPS);
        uiPartRule.setUiPart(groupListPanel);

        groupListPanelHandle = new GroupListPanelHandle(getChildNode(groupListPanel.getRoot(),
                GroupListPanelHandle.GROUP_LIST_VIEW_ID));
    }

    @Test
    public void display() {
        for (int i = 0; i < TYPICAL_GROUPS.size(); i++) {
            groupListPanelHandle.navigateToCard(TYPICAL_GROUPS.get(i));
            ReadOnlyGroup expectedGroup = TYPICAL_GROUPS.get(i);
            GroupCardHandle actualCard = groupListPanelHandle.getGroupCardHandle(i);

            assertCardDisplaysGroup(expectedGroup, actualCard);
            assertEquals(Integer.toString(i + 1) + ". ", actualCard.getId());
        }
    }

    @Test
    public void handleJumpToGroupListRequestEvent() {
        postNow(JUMP_TO_SECOND_EVENT);
        guiRobot.pauseForHuman();

        GroupCardHandle expectedCard = groupListPanelHandle.getGroupCardHandle(INDEX_SECOND_GROUP.getZeroBased());
        GroupCardHandle selectedCard = groupListPanelHandle.getHandleToSelectedCard();
        assertCardEquals(expectedCard, selectedCard);
    }
}
```
