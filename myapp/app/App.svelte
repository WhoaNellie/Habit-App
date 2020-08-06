<!-- not sure if i should have multiple habits or focus on weighing reward with ratio of goods/bads -->

<page>
    <actionBar title="Habit App" />

    <tabs tabsPosition="bottom">
            <tabStrip>
                    <tabStripItem title="To Do" />
                    <tabStripItem title="Completed" />
            </tabStrip>

            <tabContentItem>
                <gridLayout columns="*,120" rows="70,*">
                    <!-- Configures the text field and ensures that pressing Return on the keyboard
                            produces the same result as tapping the button. -->
                    <textField col="0" row="0" bind:text="{textFieldValue}" hint="Type new task..." editable="true"
                            on:returnPress="{onButtonTap}" />
                    <button col="1" row="0" text="Add task" on:tap="{onButtonTap}" class="-primary" />
            
                    <listView items="{todos}" on:itemTap="{onItemTap}" row="1" colSpan="2">
                            <Template let:item>
                                    <label text="{item.name}" textWrap="true" />
                            </Template>
                    </listView>
            </gridLayout>
            </tabContentItem>
            <tabContentItem>
                <listView items="{dones}" on:itemTap="{onDoneTap}">
                    <Template let:item>
                            <label text="{item.name}" textWrap="true" />
                    </Template>
            </listView>
            </tabContentItem>
    </tabs>
</page>

<script>
    import { Template } from 'svelte-native/components';
    import * as AppSettings from '@nativescript/core/application-settings'

    let habits;
    if(AppSettings.getString("habits")){
        habits = JSON.parse(AppSettings.getString("habits"));
    }else{
        habits = {};
        //and tutorial/setting up first habit
    }

    function addHabit(name){
        if(habits[name]){
             name = name + " the Sequel";
        }
        habits[name] = [0,0];
        AppSettings.setString("habits", JSON.stringify(habits));
    }

    function deleteHabit(name){
        delete habits[name];
        AppSettings.setString("habits", JSON.stringify(habits));
    }

    function changeHabitName(oldName, newName){
        if(habits[newName]){
             newName = newName + " the Sequel";
        }
        habits[newName] = habits[oldName];
        delete habits[oldName];
        AppSettings.setString("habits", JSON.stringify(habits));
    }

    function goodHabit(name){
        ///celebratory sound!
        habits[name][0]++;
        AppSettings.setString("habits", JSON.stringify(habits));
    }

    function badHabit(name){
        //are you sure?? checks
        //sadish sound
        habits[name][1]--;
        AppSettings.setString("habits", JSON.stringify(habits));
    }

    

    async function onEditTap(args) {
		let result = await action("What do you want to do with this task?", "Cancel", [
				"Mark completed",
				"Delete forever"
		]);

		console.log(result); // Logs the selected option for debugging.
		let item = todos[args.index]
		switch (result) {
				case "Mark completed":
						dones = addToList(dones, item) // Places the tapped active task at the top of the completed tasks.
						todos = removeFromList(todos, item) // Removes the tapped active task.
						break;
				case "Delete forever":
						todos = removeFromList(todos, item) // Removes the tapped active task.
						break;
				case "Cancel" || undefined: // Dismisses the dialog
						break;
                }
                AppSettings.setString("todos", JSON.stringify(todos));
                AppSettings.setString("dones", JSON.stringify(dones));
	}

    function onButtonTap() {
        if (textFieldValue === "") return; // Prevents users from entering an empty string.
        console.log("New task added: " + textFieldValue + "."); // Logs the newly added task in the console for debugging.
        todos = [{ name: textFieldValue }, ...todos] // Adds tasks in the ToDo array. Newly added tasks are immediately shown on the screen.
        
        AppSettings.setString("todos", JSON.stringify(todos));
        textFieldValue = ""; // Clears the text field so that users can start adding new tasks immediately.
    }

    async function onDoneTap(args) {
	let result = await action("What do you want to do with this task?", "Cancel", [
			"Mark To Do",
			"Delete forever"
	]);

	console.log(result); // Logs the selected option for debugging.
	let item = dones[args.index]
	switch (result) {
			case "Mark To Do":
					todos = addToList(todos, item) // Places the tapped active task at the top of the completed tasks.
					dones = removeFromList(dones, item) // Removes the tapped active task.
					break;
			case "Delete forever":
					dones = removeFromList(dones, item) // Removes the tapped active task.
					break;
			case "Cancel" || undefined: // Dismisses the dialog
					break;
	}

        }
</script>

<style>
	textField {
			font-size: 20;
	}
</style>
