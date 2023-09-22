---
description: >-
  Wireshark Display Filters
title:  Wireshark Display Filters           # Add title here
date: 2023-09-22 08:00:00 -0600                           # Change the date to match completion date
categories: [96 Wireless, Utilities]                     # Change Templates to Writeup
tags: [wireless, wireshark filters]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Wireshark Display Filters

Display filters are applied and edited in the Filter toolbar located between the Main toolbar and the packet list.

We will use display filters in a few different features in order to customize Wireshark and make our work easier.

Display Filter Expression
The best way to understand the syntax of Wireshark display filters is to create one with the Display Filter Expression screen. Let's select Analyze > Display Filter Expression... to open the screen with all the available filters.

Figure 11: Display Filter Expression builder
Figure 11: Display Filter Expression builder
Each display filter has a field name and relation, as well as a value, if applicable. The textbox highlighted in green is where the filter expression is displayed.

We can think of the field as an object with one or more items. The field can be built using dot-notation. This is similar to what we would see in object-oriented programming languages. The Field Name displays the object and a short description.

Hovering the mouse in the Relation window displays a pop-up that provides more details.

Figure 12: Display Filter Expression builder - Relation explanations
Figure 12: Display Filter Expression builder - Relation explanations
A display filter expression's relation can be one of the following: is present, ==, !=, <, >, >=, <=, contains, matches, or in. Depending on the field selected, not all relations will be available.

The Search field, located below the Field Name list, is useful when we can't remember a specific filter. It will narrow down the list of field names as we type, and it shows matching results from both the names and their descriptions.

Figure 13: Display Filter Expression builder - Searching for 'wlan.fc'
Figure 13: Display Filter Expression builder - Searching for 'wlan.fc'
The Predefined Values window contains a number of options that relate to different byte values in certain packet fields. For example, wlan.fc.type can have four different values: 0, 1, 2, and 3. These relate to Management, Control, Data, and Extension frames, respectively. Our chosen "wlan.fc.type == 2" filter in Figure 13 has "byte value 2", which filters for data frames.

Clicking OK updates the contents of the display filter toolbar with our selected filter and the resultant packets as shown in Figure 14.

Figure 14: Display Filter Expression builder - Applied filter 'wlan.fc.type == 2'
Figure 14: Display Filter Expression builder - Applied filter 'wlan.fc.type == 2'
Packet Details
We can also build filters based on items from a selected packet. Let's illustrate this by creating a data frame packet filter. We will start by selecting a data packet from the packet list.

Figure 15: Display Filter Expression builder - Data frame
Figure 15: Display Filter Expression builder - Data frame
In the packet's detail window, let's expand the IEEE 802.11 Data, Flags field and the Frame Control Field and then right click the Type: Data frame (2) element. Finally, we'll select Apply as Filter (selecting Analyze in the Main toolbar also displays these options).

Figure 16: Apply as Filter submenu
Figure 16: Apply as Filter submenu
Now we have a number of options to choose from.

Selected clears the display filter bar of any existing queries and generates a new query to search for this specific value. For example, wlan.fc.type with a value of "2" generates "wlan.fc.type == 2" in the display filter bar.
... and Selected appends to any existing query by using an AND (&&) condition. Taking the query we created above and adding a wlan.fc.subtype value of "0" generates "(wlan.fc.type == 2) && (wlan.fc.subtype== 0)" in the display filter bar.
... or Selected does the same as the above filter but instead of using an AND, it uses an OR (||) condition.
All the choices containing "Not" negate their equivalent in the positive form. For example, if we have "wlan.fc.type == 2" as an existing filter and use ... and not Selected with wlan.fc.subtype and value of "0", the filter becomes "(wlan.fc.type == 2) && !(wlan.fc.subtype == 0)".

Note that Apply as filter builds and applies the filter immediately. While Prepare a filter only updates the contents of the display filter bar and doesn't apply the filter until we click Apply display filter at the end of the display filter textbox.

Figure 17: Apply as Filter submenu
Figure 17: Apply as Filter submenu
Prepare a filter is useful when we want to build a complex filter that will require multiple steps. As the list of collected packets gets longer, it can take time to process each new subfilter that gets applied. It might be easier to wait to apply the filter until after we have finished writing the complete query.

Display Filter Toolbar
We can also create and access display filters directly in the Display Filter toolbar. On the left, a blue ribbon icon will bring us to bookmarked filters.

Figure 18: Display Filter bar
Figure 18: Display Filter bar
To the right of the filter bar there is an arrow and a dropdown icon. The arrow will apply the filter, while the dropdown will show the most recent display filters.

Figure 19: Display Filter bar - Recent filters dropdown
Figure 19: Display Filter bar - Recent filters dropdown
When creating a filter, Wireshark's autocompletion displays valid filters as the expression is typed.

Figure 20: Display filter autocomplete
Figure 20: Display filter autocomplete
The toolbar provides filtering hints with colored backgrounds. A valid filter is displayed with a green background. An invalid filter is indicated by a red background. For example, the "wlan.fc.type = 1" filter is syntactically incorrect because it uses only a single "=", so it appears with a red background.

Figure 21: Invalid filter
Figure 21: Invalid filter
A yellow background indicates a possibly questionable filter.

Figure 22: Filter with possibly unexpected results
Figure 22: Filter with possibly unexpected results
Yellow doesn't always mean the filter is incorrect. In most cases, it will be correct as most filters reference a single field. In this specific example, it is correct and will show packets with a wlan.fc.type with a value not equal to "1". It is good practice to use the double equal and then negate the entire expression instead. This would be written as "!(wlan.fc.type == 1)".

Display Filters Bookmarks
When doing a lot of packet filtering, we may want to reuse filters. This is where the bookmarks come in handy. They allow us to save display filters for later use. The bookmark button is located on the left of the Display Filter toolbar.

Figure 18: Display Filter bar
Figure 18: Display Filter bar
Clicking on the bookmark reveals a default set of display filters and any additional saved filters.

Figure 23: Display Filter bar
Figure 23: Display Filter bar
When a valid filter is in the Display Filter toolbar, Save this filter is clickable in the bookmark dropdown menu. Saving the filter opens the Display Filters screen and the new filter is displayed at the bottom of the list. Selecting OK saves the filter in the bookmark list.

Figure 24: Display filter window - Save filter
Figure 24: Display filter window - Save filter
We can create and save a new filter by either selecting Manage Display Filters from the bookmarks dropdown menu or via Analyze > Display Filters.... Both of these options open the Display Filters screen.

We can edit an existing filter by selecting it and applying valid changes. Valid changes are indicated by the green (or yellow) background in the Filter field. Filters from the list can be deleted with the minus ("-") button or duplicated with the Copy button.

Display Filter Buttons
We can add a shortcut to the Display Filter toolbar for frequently used filters by selecting the plus ("+") icon located on the very right of the toolbar. This opens a Create Shortcut Button panel.

Figure 25: Display Filter button settings
Figure 25: Display Filter button settings
We will enter the name of the shortcut in the Label field and enter the filter in the Filter field. The Comment field is for a description that appears when hovering the mouse over the shortcut button.

Let's create a "Data" button using our "wlan.fc.type == 2" data frames filter and an "802.11 data frames" comment.

Figure 26: Display Filter button creation
Figure 26: Display Filter button creation
Selecting OK creates our Data button to the right on the Display Filter toolbar. Hovering the mouse on the button displays the filter's comment.

Figure 27: Display Filter button
Figure 27: Display Filter button
Clicking on our new button sets the content of the filter toolbar to "wlan.fc.type == 2". Right-clicking the button gives us options to edit, disable, or remove the button.

Figure 28: Display Filter buttons preferences
Figure 28: Display Filter buttons preferences
Editing a button will bring back the creation panel below the Display Filter bar. This time, the creation panel will be prefilled with the button's existing settings.

Filter buttons can also be created, deleted, or edited via Edit > Preferences..., and selecting Filter Buttons in the left panel.

Figure 29: Display Filter buttons preferences
Figure 29: Display Filter buttons preferences

