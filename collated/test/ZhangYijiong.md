# ZhangYijiong
###### /java/seedu/address/logic/parser/AddressBookParserTest.java
``` java
    @Test
    public void parseCommand_path() throws Exception {
        PathCommand command = (PathCommand) parser.parseCommand(
                PathCommand.COMMAND_WORD + " " + INDEX_SECOND_PERSON.getOneBased());
        assertEquals(new PathCommand(INDEX_SECOND_PERSON), command);
    }

```
###### /java/seedu/address/logic/parser/AddressBookParserTest.java
``` java
    @Test
    public void parseCommand_pathAlias() throws Exception {
        PathCommand command = (PathCommand) parser.parseCommand(
                PathCommand.COMMAND_ALIAS + " " + INDEX_SECOND_PERSON.getOneBased());
        assertEquals(new PathCommand(INDEX_SECOND_PERSON), command);
    }

```
###### /java/seedu/address/logic/parser/PathCommandParserTest.java
``` java
package seedu.address.logic.parser;

import static seedu.address.commons.core.Messages.MESSAGE_INVALID_COMMAND_FORMAT;
import static seedu.address.logic.parser.CommandParserTestUtil.assertParseFailure;
import static seedu.address.logic.parser.CommandParserTestUtil.assertParseSuccess;
import static seedu.address.testutil.TypicalIndexes.INDEX_THIRD_PERSON;

import org.junit.Test;

import seedu.address.logic.commands.PathCommand;

/**
 * Test scope: similar to {@code DeleteCommandParserTest} and to{@code SelectCommandParserTest}}.
 * @see DeleteCommandParserTest
 */
public class PathCommandParserTest {
    private PathCommandParser parser = new PathCommandParser();

    @Test
    public void parse_validArgs_returnsPathCommand() {
        assertParseSuccess(parser, "3", new PathCommand(INDEX_THIRD_PERSON));
    }

    @Test
    public void parse_invalidArgs_throwsParseException() {
        assertParseFailure(parser, "ABC", String.format(MESSAGE_INVALID_COMMAND_FORMAT, PathCommand.MESSAGE_USAGE));
    }
}
```
###### /java/seedu/address/logic/commands/PathCommandTest.java
``` java
package seedu.address.logic.commands;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;
import static org.junit.Assert.fail;
import static seedu.address.logic.commands.CommandTestUtil.showPersonAtIndex;
import static seedu.address.testutil.TypicalIndexes.INDEX_FIRST_PERSON;
import static seedu.address.testutil.TypicalIndexes.INDEX_SECOND_PERSON;
import static seedu.address.testutil.TypicalIndexes.INDEX_THIRD_PERSON;
import static seedu.address.testutil.TypicalPersons.getTypicalAddressBook;

import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;

import seedu.address.commons.core.Messages;
import seedu.address.commons.core.index.Index;
import seedu.address.commons.events.ui.PersonPanelPathChangedEvent;
import seedu.address.logic.CommandHistory;
import seedu.address.logic.UndoRedoStack;
import seedu.address.logic.commands.exceptions.CommandException;
import seedu.address.model.CustomerStats;
import seedu.address.model.Menu;
import seedu.address.model.Model;
import seedu.address.model.ModelManager;
import seedu.address.model.UserPrefs;
import seedu.address.ui.testutil.EventsCollectorRule;

/**
 * is almost the same as {@code SelectCommandTest}
 * Contains integration tests (interaction with the Model) for {@code SelectCommand}.
 */
public class PathCommandTest {
    @Rule
    public final EventsCollectorRule eventsCollectorRule = new EventsCollectorRule();

    private Model model;

    @Before
    public void setUp() {
        model = new ModelManager(getTypicalAddressBook(), new UserPrefs(), new CustomerStats(), new Menu());
    }

    /**
     * is the same to {@code SelectCommandTest}
     * is not able to work due to unsolved problem during initialization of javafx.fxml in
     * initialization of PersonCard in execution of PathCommand in assertExecutionSuccess
     */
    public void execute_validIndexUnfilteredList_success() {
        Index lastPersonIndex = Index.fromOneBased(model.getFilteredPersonList().size());

        assertExecutionSuccess(INDEX_FIRST_PERSON);
        assertExecutionSuccess(INDEX_THIRD_PERSON);
        assertExecutionSuccess(lastPersonIndex);
    }

    /**
     * is the same to {@code SelectCommandTest}
     * is not able to work due to unsolved problem during initialization of javafx.fxml in
     * initialization of PersonCard in execution of PathCommand in assertExecutionSuccess
     */
    public void execute_validIndexFilteredList_success() {
        showPersonAtIndex(model, INDEX_FIRST_PERSON);

        assertExecutionSuccess(INDEX_FIRST_PERSON);
    }

    @Test
    public void execute_invalidIndexUnfilteredList_failure() {
        Index outOfBoundsIndex = Index.fromOneBased(model.getFilteredPersonList().size() + 1);

        assertExecutionFailure(outOfBoundsIndex, Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
    }

    @Test
    public void execute_invalidIndexFilteredList_failure() {
        showPersonAtIndex(model, INDEX_FIRST_PERSON);

        Index outOfBoundsIndex = INDEX_SECOND_PERSON;
        // ensures that outOfBoundIndex is still in bounds of address book list
        assertTrue(outOfBoundsIndex.getZeroBased() < model.getAddressBook().getPersonList().size());

        assertExecutionFailure(outOfBoundsIndex, Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
    }

    @Test
    public void equals() {
        PathCommand pathFirstCommand = new PathCommand(INDEX_FIRST_PERSON);
        PathCommand pathSecondCommand = new PathCommand(INDEX_SECOND_PERSON);
        PathCommand pathThirdCommand = new PathCommand(INDEX_THIRD_PERSON);

        // same object -> returns true
        assertTrue(pathFirstCommand.equals(pathFirstCommand));
        assertTrue(pathThirdCommand.equals(pathThirdCommand));

        // same values -> returns true
        PathCommand pathSecondCommandCopy = new PathCommand(INDEX_SECOND_PERSON);
        assertTrue(pathSecondCommand.equals(pathSecondCommandCopy));

        // different types -> returns false
        assertFalse(pathFirstCommand.equals(1));

        // null -> returns false
        assertFalse(pathSecondCommand.equals(null));

        // different person -> returns false
        assertFalse(pathFirstCommand.equals(pathThirdCommand));
    }

    /**
     * Executes a {@code PathCommand} with the given {@code index}, and checks that {@code PersonPanelPathChangedEvent}
     * is similar to {@code SelectCommandTest} except for Event
     * is raised with the correct index.
     */
    private void assertExecutionSuccess(Index index) {
        PathCommand pathCommand = prepareCommand(index);

        try {
            CommandResult commandResult = pathCommand.execute();
            assertEquals(String.format(PathCommand.MESSAGE_PATH_PERSON_SUCCESS, index.getOneBased()),
                    commandResult.feedbackToUser);
        } catch (CommandException ce) {
            throw new IllegalArgumentException("Execution of command should not fail.", ce);
        }

        PersonPanelPathChangedEvent lastEvent =
                (PersonPanelPathChangedEvent) eventsCollectorRule.eventsCollector.getMostRecent();
        assertEquals(index, Index.fromZeroBased(lastEvent.getNewSelection().getDisplayedIndex()));
    }

    /**
     * Executes a {@code SelectCommand} with the given {@code index}, and checks that a {@code CommandException}
     * is the same to {@code SelectCommandTest} except for the test object
     * is thrown with the {@code expectedMessage}.
     */
    private void assertExecutionFailure(Index index, String expectedMessage) {
        PathCommand pathCommand = prepareCommand(index);

        try {
            pathCommand.execute();
            fail("The expected CommandException was not thrown.");
        } catch (CommandException ce) {
            assertEquals(expectedMessage, ce.getMessage());
            assertTrue(eventsCollectorRule.eventsCollector.isEmpty());
        }
    }

    /**
     * Returns a {@code PathCommand} with parameters {@code index}.
     */
    private PathCommand prepareCommand(Index index) {
        PathCommand pathCommand = new PathCommand(index);
        pathCommand.setData(model, new CommandHistory(), new UndoRedoStack());
        return pathCommand;
    }
}
```
###### /java/seedu/address/logic/commands/AddOrderCommandTest.java
``` java
package seedu.address.logic.commands;

import static java.util.Objects.requireNonNull;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;
import static org.junit.Assert.fail;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.function.Predicate;

import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ExpectedException;

import javafx.collections.ObservableList;
import seedu.address.logic.CommandHistory;
import seedu.address.logic.UndoRedoStack;
import seedu.address.logic.commands.exceptions.CommandException;
import seedu.address.model.AddressBook;
import seedu.address.model.Model;
import seedu.address.model.ReadOnlyAddressBook;
import seedu.address.model.dish.exceptions.DishNotFoundException;
import seedu.address.model.person.Person;
import seedu.address.model.person.exceptions.DuplicatePersonException;
import seedu.address.model.person.exceptions.PersonNotFoundException;
import seedu.address.model.task.Task;
import seedu.address.model.task.exceptions.DuplicateTaskException;
import seedu.address.model.task.exceptions.TaskNotFoundException;
import seedu.address.testutil.TaskBuilder;

/**
 * Implementation follows {@code AddCommandTest}
 */

public class AddOrderCommandTest {

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Test
    public void constructor_nullTask_throwsNullPointerException() {
        thrown.expect(NullPointerException.class);
        new AddOrderCommand(null);
    }

    @Test
    public void execute_taskAcceptedByModel_addSuccessful() throws Exception {
        ModelStubAcceptingTaskAdded modelStub = new ModelStubAcceptingTaskAdded();
        Task validTask = new TaskBuilder().build();

        CommandResult commandResult = getAddOrderCommandForTask(validTask, modelStub).execute();

        assertEquals(String.format(AddOrderCommand.MESSAGE_SUCCESS, validTask), commandResult.feedbackToUser);
        assertEquals(Arrays.asList(validTask), modelStub.tasksAdded);
    }

    @Test
    public void execute_duplicateTask_throwsCommandException() throws Exception {
        ModelStub modelStub = new ModelStubThrowingDuplicateTaskException();
        Task validTask = new TaskBuilder().build();

        thrown.expect(CommandException.class);
        thrown.expectMessage(AddOrderCommand.MESSAGE_DUPLICATE_TASK);

        getAddOrderCommandForTask(validTask, modelStub).execute();
    }

    @Test
    public void equals() {
        Task chickenRice = new TaskBuilder().withOrder("Chicken Rice").build();
        Task charSiuRice = new TaskBuilder().withOrder("Char Siu Rice").build();
        AddOrderCommand addOrderChickenRiceCommand = new AddOrderCommand(chickenRice);
        AddOrderCommand addOrderCharSiuRiceCommand = new AddOrderCommand(charSiuRice);

        // same object -> returns true
        assertTrue(addOrderChickenRiceCommand.equals(addOrderChickenRiceCommand));

        // same values -> returns true
        AddOrderCommand addOrderChickenRiceCommandCopy = new AddOrderCommand(chickenRice);
        assertTrue(addOrderChickenRiceCommand.equals(addOrderChickenRiceCommand));

        // different types -> returns false
        assertFalse(addOrderChickenRiceCommand.equals(1));

        // null -> returns false
        assertFalse(addOrderChickenRiceCommand.equals(null));

        // different task -> returns false
        assertFalse(addOrderChickenRiceCommand.equals(addOrderCharSiuRiceCommand));
    }

    /**
     * Generates a new AddOrderCommand with the details of the given task.
     */
    private AddOrderCommand getAddOrderCommandForTask(Task task, Model model) {
        AddOrderCommand command = new AddOrderCommand(task);
        command.setData(model, new CommandHistory(), new UndoRedoStack());
        return command;
    }

    /**
     * A default model stub that have all of the methods failing.
     */
    private class ModelStub implements Model {
        @Override
        public void addPerson(Person person) throws DuplicatePersonException {
            fail("This method should not be called.");
        }

        @Override
        public void resetData(ReadOnlyAddressBook newData) {
            fail("This method should not be called.");
        }

        @Override
        public ReadOnlyAddressBook getAddressBook() {
            fail("This method should not be called.");
            return null;
        }

        @Override
        public void deletePerson(Person target) throws PersonNotFoundException {
            fail("This method should not be called.");
        }

        @Override
        public void updatePerson(Person target, Person editedPerson)
                throws DuplicatePersonException {
            fail("This method should not be called.");
        }

        @Override
        public ObservableList<Person> getFilteredPersonList() {
            fail("This method should not be called.");
            return null;
        }

        @Override
        public void updateFilteredPersonList(Predicate<Person> predicate) {
            fail("This method should not be called.");
        }

        @Override
        public void checkOrder(Person target) throws DishNotFoundException {
            fail("This method should not be called.");
        }

        @Override
        public void addTask(Task task) throws DuplicateTaskException {
            fail("This method should not be called.");
        }

        //@Override deleteTask has not been implemented yet
        public void deleteTask(Task target) throws TaskNotFoundException {
            fail("This method should not be called.");
        }

        @Override
        public ObservableList<Task> getFilteredTaskList() {
            fail("This method should not be called.");
            return null;
        }

        @Override
        public void updateFilteredTaskList(Predicate<Task> predicate) {
            fail("This method should not be called");
        }
```
###### /java/seedu/address/logic/commands/AddCommandTest.java
``` java
        @Override
        public void addTask(Task task) throws DuplicateTaskException {
            fail("This method should not be called.");
        }

        @Override
        public ObservableList<Task> getFilteredTaskList() {
            fail("This method should not be called.");
            return null;
        }

        @Override
        public void updateFilteredTaskList(Predicate<Task> predicate) {
            fail("This method should not be called");
        }
```
###### /java/seedu/address/testutil/TaskBuilder.java
``` java
package seedu.address.testutil;

import seedu.address.model.dish.Price;
import seedu.address.model.person.Address;
import seedu.address.model.person.Order;
import seedu.address.model.task.Count;
import seedu.address.model.task.Distance;
import seedu.address.model.task.Task;

/**
 * Implementation follows {@code PersonBuilder}
 * A utility class to help with building Task objects.
 */
public class TaskBuilder {

    public static final String DEFAULT_TASK_ORDER = "Chicken Rice";
    public static final String DEFAULT_TASK_ADDRESS = "1A Kent Ridge Road";
    public static final String DEFAULT_TASK_PRICE = "100";
    public static final String DEFAULT_TASK_DISTANCE = "5";
    public static final String DEFAULT_TASK_COUNT = "2";
    public static final String DEFAULT_DESCRIPTION = "Chili Sauce Required";

    private Order defaultOrder;
    private Address defaultAddress;
    private Price defaultPrice;
    private Distance defaultDistance;
    private Count defaultCount;
    private String defaultDescription;

    public TaskBuilder()  {
        defaultOrder = new Order(DEFAULT_TASK_ORDER);
        defaultAddress = new Address(DEFAULT_TASK_ADDRESS);
        defaultPrice = new Price(DEFAULT_TASK_PRICE);
        defaultDistance = new Distance(DEFAULT_TASK_DISTANCE);
        defaultCount = new Count(DEFAULT_TASK_COUNT);
        defaultDescription = DEFAULT_DESCRIPTION;
    }

    /**
     * Initializes the TaskBuilder with the data of {@code taskToCopy}.
     */
    public TaskBuilder(Task taskToCopy) {
        defaultOrder = taskToCopy.getOrder();
        defaultAddress = taskToCopy.getAddress();
        defaultPrice = taskToCopy.getPrice();
        defaultDistance = taskToCopy.getDistance();
        defaultCount = taskToCopy.getCount();
        defaultDescription = taskToCopy.getDescription();
    }

    /**
     * Sets the {@code Order} of the {@code Task} that we are building.
     */
    public TaskBuilder withOrder(String order) {
        this.defaultOrder = new Order(order);
        return this;
    }

    /**
     * Sets the {@code Address} of the {@code Task} that we are building.
     */
    public TaskBuilder withAddress(String address) {
        this.defaultAddress = new Address(address);
        return this;
    }

    /**
     * Sets the {@code Price} of the {@code Task} that we are building.
     */
    public TaskBuilder withPrice(String price) {
        this.defaultPrice = new Price(price);
        return this;
    }

    /**
     * Sets the {@code Distance} of the {@code Task} that we are building.
     */
    public TaskBuilder withDistance(String distance) {
        this.defaultDistance = new Distance(distance);
        return this;
    }

    /**
     * Sets the {@code Count} of the {@code Task} that we are building.
     */
    public TaskBuilder withCount(String count) {
        this.defaultCount = new Count(count);
        return this;
    }

    /**
     * Sets the {@code Description} of the {@code Task} that we are building.
     */
    public TaskBuilder withDescription(String description) {
        this.defaultDescription = description;
        return this;
    }

    /**
     * build task
     */
    public Task build() {
        return new Task(defaultOrder, defaultAddress, defaultPrice,
                defaultDistance, defaultCount, defaultDescription);
    }
}

```
###### /java/seedu/address/testutil/TaskUtil.java
``` java
package seedu.address.testutil;

import static seedu.address.logic.parser.CliSyntax.PREFIX_ADDRESS;
import static seedu.address.logic.parser.CliSyntax.PREFIX_COUNT;
import static seedu.address.logic.parser.CliSyntax.PREFIX_DESCRIPTION;
import static seedu.address.logic.parser.CliSyntax.PREFIX_DISTANCE;
import static seedu.address.logic.parser.CliSyntax.PREFIX_ORDER;
import static seedu.address.logic.parser.CliSyntax.PREFIX_PRICE;

import seedu.address.logic.commands.AddOrderCommand;
import seedu.address.model.task.Task;

/**
 * Implementation follows {@code PersonUtil}
 * A utility class for Task.
 */
public class TaskUtil {

    /**
     * Returns an addOrder command string for adding the {@code task}.
     */
    public static String getAddOrderCommand(Task task) {
        return AddOrderCommand.COMMAND_WORD + " " + getTaskDetails(task);
    }

    /**
     * Returns the part of command string for the given {@code task}'s details.
     */
    public static String getTaskDetails(Task task) {
        StringBuilder sb = new StringBuilder();
        sb.append(PREFIX_ORDER + task.getOrder().toString() + " ");
        sb.append(PREFIX_ADDRESS + task.getAddress().value + " ");
        sb.append(PREFIX_PRICE + task.getPrice().value + " ");
        sb.append(PREFIX_DISTANCE + task.getDistance().value + " ");
        sb.append(PREFIX_COUNT + task.getCount().value + " ");
        sb.append(PREFIX_DESCRIPTION + task.getDescription() + " ");

        return sb.toString();
    }
}

```