# Sudoku by Kapil Easwar

## How to run
1. Clone the git repo (see instructions in Github)
2. cd sudoku (into the repo)
3. run 'node web.js'
4. The server will now be listening on localhost:8080

## Technical design
### Server: web.js
The server is very thin. It simply responds to requests to '/' and returns a page that
has the hardcoded Sudoku board

### Client
The client side uses an MVC pattern as much as possible
- Model: public/SudokuModel.js, url: '/public/SudokuModel.js'
- Controller: public/SudokuController.js, url: '/public/SudokuController.js'
- View: sudoku.jade, url: '/'
    
The styling uses LESS to compile the CSS and use some variables for values that may change
        less file: public/pages/sudoku.less, compiled to a CSS file by the 'less-middleware' module

#### Parts of the client
- *MODEL: SudokuModel.js*
        This object initializes by taking in the board data generated by the server. It holds the data on the client
        and has an API to get values, set values, determine which values are given, and calculate the validity
        of squares. It is the source of truth of the state of the board. It also supports having a callback that
        is useful for being notified of changes to the model

- *CONTROLLER: SudokuController.js*
        This object connects the model to the view. It sets up handlers for when the model changes, to update the
        view. It also puts handlers on parts of the UI (the view) to update the model. (It avoids an infinite loop
        by not firing the 'change' event when the UI is updated to reflect the model.) There's a little inefficiency
        because when you update the UI, it updates the model which updates the UI again, but that's necessary to
        keep the UI in sync when the user tries to change a given square

- *VIEW: sudoku.jade and sudoku.less*
    The Sudoku board is a simple CSS table with borders. The input panel is an array of number, which when clicked
        will put that number in the currently-selected square (shaded with gray). You can also click a square and
        type a number directly into it (this is for ease of entry on computer, while the buttons are more for mobile)

    This input method was chosen because, unlike a pop-up, it doesn't hide the board as you're entering a value. The
        keyboard input method is more for keyboard-based inputs

## Things I would improve
- Remove duplication of constants
    I tried to make it as extensible as possible to when its a 4x4x4 board, for example. The problem is, these dimensions
    are in constants all over the place. In the server in web.js, and two places in the client in SudokuModel.js and
    SudokuController.js. This could be solved by simply storing it on the server, sending it over with the board data
    and putting a getSize() method in the model, which everthing else references

- Use Jade mixins
    I would use mixins to make reusable bits of the UI. The homepage would just include a sudokuBoard.jade mixin which handled
    initializing the model and controller given the board data. The sudokuBoard.jade mixin would use a mixin for each square,
    so the markup on each square would be separated into its own file

    This would then also allow each mixin to add the references to the JS and CSS files it needed. This would require them
    to declare dependencies that get injected once into the <head> of the page (once so that multiple instances of the
    same mixin wouldn't request the required scripts and CSS multiple times)

- Allow specifying a port
    This is a simple one: the port should be defined in an environment variable. I was going to do this so it could be
    hosted on Heroku, but I lost my two-factor auth codes :( so I decided not to bother. I could have done it so it could
    be run with foreman locally, but I wanted to lower the barrier for you to be able to run it (just need node and npm installed)

- Accessibility
    This simple grid-based UI can and should be accessible, meaning using the right aria-* attributes, and adding a keyboard
    interface that allows the user to cycle through squares on the board with the arrow keys instead of tabbing

- Mobile UI and other UI polish
    The input buttons are still a bit too small on mobile
    The font size also should scale a bit better. It's a little small on mobile, and when you zoom in/out the text should
        scale with the squares of the sudoku board
    Also, I'd use better colors and borders

## Design choices and scoping decisions
- Not storing the solution
    I decided not to store the board's solution mostly to cut scope and time. This means that all the validation
    (when the user clicks "Check") is only based on "visible" errors. If the user enters a 2 in a square, the validation
    will only mark it invalid if there's another 2 in the row/col/sub-square, even if the correct solution has a 4 there.
    Because of this, I'm also not able to provide any hints or anything like that

- Not disabling the input buttons when there's no square selected

- Not using icons for the "Clear" and "Check" buttons. Furthermore, the "Clear" button is confusing because it might look
    like it clears the whole board. I decided not to expose that function, so the user just has to refresh the page.

- Validating "given" squares
    When validating, "given" squares will get highlighted too even though you can't change them. This is because it
    helps highlight what makes the other one invalid

- After you select a value for a square, maybe the focus should jump back to it. But I decided not to do that because the
    UX might be a little weird

- I'm not allowing user to validate just one square, because I didn't know what the UX would look like for that. Instead
    they have to validate the whole board at once

## Other scoping decisions/future tasks
1. Display error message when user inputs something invalid, or tries to change a given square
2. Display message when user validates and there are no problems
3. Display a success message when the board is finished
4. Add hover text for buttons to explain what they do
5. Hide or disable the input panel when there is no selected square
6. Allow for multiple onChange callbacks in the model
7. Only have pop-up when navigating away from the page when there are actually unsaved changes
8. Allow for undo/redo
9. Actually generate Sudoku board
10. Store the stored Sudoku board in the session so it's not lost on page refresh. Provide a "New game" button to
        explicitly request a new board
11. Add a timer and user profiles so a user can track progress and compare to their friends
