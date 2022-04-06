import PySimpleGUI as sg
# Library that does all the graphics interface.
# In order to use it, you need to install PySimpleGUI before launching the program
# The command for installing PySimpleGUI in Windows is "python -m pip install PySimpleGUI" (to be run in cmd with admin privileges)

# Node class - Handles logic of nodes in the state space graph
class Node:
#The class node, we define a type that is a node in the state space graph.
#And everything that is inside of this, is particular to that node type.


      #constructor
      def __init__(self, is_max, standing):
            #When we define a new node, whether is min or max, and what's the current standing of the game.
            #That's how we define game states. Because the game states here are the amount of matches left, and also if it's a Min or Max node.
            self.max = is_max #if it's Min or Max, it's a boolean. If true it's a Max, if false is a Min.
            self.children = [] #are all the states that can be reached from this game state. These are child nodes
            self.value = 0 #value is either negative one or positive one, depending on if it loeads to win or loss.
            self.standing = standing #it's the current game state, what matches are still available in the game

      def addChild(self, child):
            self.children.append(child)
            # Simply adds a child node, a reference to a game state that can be reached from this game state.
            # We put a child at the end of the children list.


      #calculate the value of a note depending children and whether it is min or max
      def calcValue(self):
            if self.value == 0:     #only calculate if not set already
                  #We start the value at 0, and the value at the end can only be either 1 or -1, so if it's still 0, 
                  # we know we haven't set it yet, and therefore we need to calculate it.

                  values = [] #it's the values of all the child nodes, we define it in order to fill it later on.
                  if len(self.children) > 0:
                        
                        for c in self.children:
                              values.append(c.getValue())
                              #if there are any children there, we loop over all the children, and we just put the value of all the children in this value field
                        if self.max:
                              self.value = max(values)
                              # Then, if the node is a Max node, the program should take the Max value of all the children, 
                              # therefore we can return the Max value of the other value we aggregated, 
                              # and if it's a Min node, we just return the minimum value.
                        else:
                              self.value = min(values)

      #calculate value and return afterwards
      def getValue(self):
            # Here, with getValue, we get the value of that specific node we are currently dealing with in the state space graph.
            self.calcValue()
            return self.value

      def setValue(self, input):
            # This is simply used for the nodes that are at the very bottom, so we can set the value of these.
            # Because we are checking if we win or we lose, therefore we set it to 1 or -1.
            self.value = input

      # Return an advantageous move from the current standing
      def getMove(self):
            # We want to get what should be done by the computer, in order to win the game. So we loop over all the children, 
            # all the states that can be reached from that state, all the possible moves. 
            # And if that's a Max node, we return the first following state, that results in a positive or winning.
            for c in self.children:
                  if self.max == True:
                        if c.getValue() == 1:
                              return c
                  else:
                        #if it's a Min node, we return the first child that has value of -1 
                        if c.getValue() == -1:
                              return c
            return self.children[0] # This is in case if there are no children with these values (-1 or 1). 
            # This is placed just for theoretical failure that might occur (prevent return of NULL).


      def getChildren(self): # This returns all the children that were assigned. In order to check what the children are.
            return self.children

      def getStanding(self): # In order to check what the standing is.
            return self.standing
      
      def isMax(self): # In order to check if it's Min or Max.
            return self.max


#init CI variables
max_matches = [1,2,3]
in_standing = max_matches
state_space = []
root_p = Node(False, [0,0,0])
root_c = Node(True, [0,0,0])
player_move = [0,0,0]
current_state = root_p

#Fill list with all possible game states
for i in range(0,max_matches[0] + 1):
      # These are 3 nested for loops, in this way we go over all the possible states. 
      # We begin at (0, 0, 0), that is the state where all matches are gone, and then we put in "state_space" (list) a node where the player did the last move,
      # and where the computer did the last move. So one is Min and one is Max, and the we add to the list.
      # Then we increment the k, and we have (0, 0, 1), (0, 0, 2) etc, until we reach for example 3, that is the maximum number of matches in that specific row. 
      # After we have added the node, j is incremented and k is started from 0 again. 
      # In this way, we fill all the possible combinations of matches, and we have every possible combination.
      for j in range(0,max_matches[1] +1):
            for k in range(0,max_matches[2] + 1):
                  state_space.append(Node(True, [i,j,k]))
                  state_space.append(Node(False, [i,j,k]))


#Setup state space graph
for s in state_space:
      # After we went over all the possible states, we need to link them togheter to get the state space graph,
      # from which we can see what state can we reach from the specific state we are in a specific turn.
      # We loop over all the possible states, we get the standing, and if the standing is MaxMatches, 
      # this means it is a starting state, this is a root node, so whether the game is new and all the matches are still there.
      # And if it's Max, we set it to "root_c", that stands for the start where the computer does the first move, 
      # while "root_p" is where at the start the player does the first move.
      
      stand = s.getStanding()

      #If standing of state matches initial standing, make it a root node
      if stand == max_matches:
            if s.isMax():     #isMax indicates wether a node is min or max but also if the player(min) or computer(max) does the next move
                  root_c = s
            else:
                  root_p = s

      # If the sum of the standings is 0 (0, 0 ,0) , this means that the last one that did the move is the loser,
      # because they took the last match. If it's Max, the player did the last move, we set it to 1 because we (the computer) win.
      # While, if it's not Max we (the computer) did the last move, and we set it to -1 because we (the computer) lost.

      if sum(stand) == 0 and s.isMax():   #If no match remains and the player performed the move leading to this, computer wins (value set to 1)
            s.setValue(1)
      elif sum(stand) == 0 and not s.isMax():   #If no match remains and the computer performed the move leading to this, computer loses (value set to -1)
            s.setValue(-1)
      elif sum(stand) == 1 and s.isMax():   #If one match remains and the player performed the move leading to this, computer loses (value set to -1)
            # here we have to do the exact opposite, because if the player did the last move, it would mean that we (the computer) 
            # would have to take the last match, and therefore we (the computer) lose.
            s.setValue(-1)
      elif sum(stand) == 1 and not s.isMax():   #If one match remains and the computer performed the move leading to this, computer wins (value set to 1)
            # Also, if the status is 1 and we did the last move, we (the computer) have won so we set the value of the node to 1.
            s.setValue(1)

      #Connect state of the state space graph
      for c in state_space:
            c_stand = c.getStanding()
            # For every node we check what are the other possible child nodes, so we iterate again over all the states of the state space graph, 
            # and then just check if the state we are proposing as the parent state, and the state we are proposing as the child state, 
            # so the state that can be reached from there, they have to be alternating. 
            # So, you get in your graph a Max node and that points to only Min nodes, and those only point to Max nodes, 
            # and it always swtiches, so that they are not the same.
            # And the child should have smaller value in the standing than the parent, because when you do a move, 
            # there's always one match that goes away, so the standing has to decrease. 
            # If that's the case, then it's a possible child, then we have to check only if matches from one row were removed.

            if not (s.isMax() ==  c.isMax()) and sum(stand) > sum(c_stand):     # Propose child if the actor switches and sum of child is lower than sum of parent
                  if stand[0] > c_stand[0] and stand[1] == c_stand[1] and stand[2] == c_stand[2]:    # select child if only first row changed
                        # If we remove one from the first row, but not from the second and third, we add the child.
                        s.addChild(c)
                  elif stand[0] == c_stand[0] and stand[1] > c_stand[1] and stand[2] == c_stand[2]:      # select child if only second row changed
                        # We removed it only from the second row (not first and third rows), we add the child.
                        s.addChild(c)
                  elif stand[0] == c_stand[0] and stand[1] == c_stand[1] and stand[2] > c_stand[2]:      # select child if only third row changed
                        # We removed it only from the third row (not first and second rows), we add the child.
                        s.addChild(c)

            # In any other case, if there's one from the first missing, and one from the second missing,
            # this is not a possible state that can be reached, therefore we don't put it as child.


root_p.getValue()
root_c.getValue()
# These are just to preprocess the values, because this is recursive, we generate the value, by getting the value of the children. 
# And everytime you get a value, the value of the node is calculated, therefore we calculate down for the entire space state graph. 
# Once the calculations are done, the values of all nodes are set and can be accessed very quickly.



#find the child state that matches the new standing
def findState(n : Node, standing):
      for c in n.getChildren():
            if c.getStanding() == standing:
                  return c
      print("Returning n")
      return n

#get the computers move according to the current game state
def getMove(c_state,standing):
      st = findState(c_state, standing).getMove()
      return [st, [standing[0] - st.getStanding()[0], standing[1] - st.getStanding()[1], standing[2] - st.getStanding()[2]]]

def performCompMove():
      global current_state
      global in_standing
      global started

      [current_state, res] = getMove(current_state, in_standing)

      #Update standing
      in_standing = current_state.getStanding()

      #Hide matches, that were removed during the computer move
      for n in range(0, res[0]):
            window.find_element('match1.' + str(row1[0])).update(visible=False)
            row1.remove(row1[0])
      for n in range(0, res[1]):
            window.find_element('match2.' + str(row2[0])).update(visible=False)
            row2.remove(row2[0])
      for n in range(0, res[2]):
            window.find_element('match3.' + str(row3[0])).update(visible=False)
            row3.remove(row3[0])
      
      #If computer makes game ending move, show menu items
      if sum(in_standing) == 1:
            window.find_element("lossc").unhide_row()
            showMenu1()
            started = False

def hideMatches():
      window.find_element("match1c").hide_row()
      window.find_element("match2c").hide_row()
      window.find_element("match3c").hide_row()

def showMatches():
      window.find_element("match1c").unhide_row()
      window.find_element("match2c").unhide_row()
      window.find_element("match3c").unhide_row()

def showMenu1():
      window.find_element("menu1c").unhide_row()
      window.find_element("menu2c").unhide_row()
      window.find_element("end_turnc").hide_row()

def hideMenu1():
      window.find_element("menu1c").hide_row()
      window.find_element("menu2c").hide_row()

def showMenu2():
      window.find_element("rules").unhide_row()
      window.find_element("menu3c").unhide_row()
      window.find_element("menu4c").unhide_row()

def hideMenu2():
      window.find_element("rules").hide_row()
      window.find_element("menu3c").hide_row()
      window.find_element("menu4c").hide_row()

#place UI element in column to keep position, when making invisible
def placeL(elem, k):
    return sg.Column([elem], pad=(0,0), justification='center', key = k)

#place list of UI elements in column to keep position and make centered
def placeE(elem, k):
    return sg.Column([[elem]], pad=(0,0), justification='center', key = k)

#init UI variables
match_row_1 = []
match_row_2 = []
match_row_3 = []
row1 = []
row2 = []
row3 = []
started = False

#fill rows with matches
for i in range(max_matches[0]):
      match_row_1.append(placeE(sg.Image('python.png', key = 'match1.' + str(i),  size=(30,200), subsample = 3, enable_events = True), "m1"))
      row1.append(i)
for i in range(max_matches[1]):
      match_row_2.append(placeE(sg.Image('python.png', key = 'match2.' + str(i),  size=(30,200), subsample = 3, enable_events = True), "m2"))
      row2.append(i)
for i in range(max_matches[2]):
      match_row_3.append(placeE(sg.Image('python.png', key = 'match3.' + str(i),  size=(30,200), subsample = 3, enable_events = True), "m3"))
      row3.append(i)

#assemble UI layout
layout = [[sg.Text("Welcome to Nim", size=(50,2), justification='center')], 
[placeE(sg.Text("Rules:\nThis games is a turn-based.\nYou are allowed to take any amount of matches you wish, as long as they're from the same row.\nOnce your turn is over, click on 'End turn'.\n You must pick at least one match before ending the turn. \nThe one who picks the last match, loses the game.", size=(50,6), justification='center'),"rules")], 
[placeE(sg.Button("End turn", key = "end_turn", size =(30,3)), "end_turnc")],
[placeE(sg.Button("Player does the first move", key = "menu3",size =(40,3), visible = True), "menu3c")],
[placeE(sg.Button("Computer does the first move", key = "menu4",size =(40,3), visible = True), "menu4c")],
[placeE(sg.Text("Remove at least one match to make a move", key = "notice", size=(50,2), justification='center'), "noticec")],
[placeL(match_row_1, "match1c")],
[placeL(match_row_2, "match2c")],
[placeL(match_row_3, "match3c")],
[placeE(sg.Text("You have won! Do you want wo play another game ?", key = "win", size=(50,2), justification='center'), "winc")],
[placeE(sg.Text("You have lost! Do you want wo play another game ?", key = "loss",  size=(50,2), justification='center'), "lossc")],
[placeE(sg.Button("Play again", key = "menu1",size =(40,3)), "menu1c")],
[placeE(sg.Button("Exit game", key = "menu2",size =(40,3)), "menu2c")]
]

# Create the window
window = sg.Window("Nim", layout)

window.finalize()

window.find_element("winc").hide_row()
window.find_element("lossc").hide_row()
window.find_element("noticec").hide_row()
window.find_element("end_turnc").hide_row()
hideMenu1()
hideMatches()

window.move(700, 100)


# Create an event loop
while True:
      #wait for user input
      event, values = window.read()

      #close window, when 'End Game' is clicked
      if event == "menu2" or event == sg.WIN_CLOSED:
            break

      #Reset game, when 'Play Again' is clicked
      if event == "menu1":

            #Reset player move and initial standing
            player_move = [0,0,0]
            in_standing = max_matches

            #Clear index lists for matches
            row1.clear()
            row2.clear()
            row3.clear()

            #Refill index lists for matches
            for j in range(0,len(match_row_1)):
                  row1.append(j)
            for j in range(0,len(match_row_2)):
                  row2.append(j)
            for j in range(0,len(match_row_3)):
                  row3.append(j)

            #Make all matches visible
            for j in range(0,len(match_row_1)):
                 window.find_element('match1.' + str(j)).update(visible=True)
            for j in range(0,len(match_row_2)):
                  window.find_element('match2.' + str(j)).update(visible=True)
            for j in range(0,len(match_row_3)):
                  window.find_element('match3.' + str(j)).update(visible=True)

            #Make menu items invisible again
            window.find_element("winc").hide_row()
            window.find_element("lossc").hide_row()
            hideMenu1()
            hideMatches()
            showMenu2()

      if event == "menu3":
            hideMenu2()
            window.find_element("end_turnc").unhide_row()
            showMatches()
            current_state = root_p
            started =True
      
      if event == "menu4":
            hideMenu2()
            window.find_element("end_turnc").unhide_row()
            showMatches()
            current_state = root_c
            started = True
            performCompMove()

            
      #Handle player move, game state change and computer move when 'End turn' is clicked
      if event == "end_turn":

            #Check if player removed at least one match
            if sum(player_move) == 0:
                  #Show message (Remove at least one ...)
                  window.find_element("noticec").unhide_row()
            else:
                  #Unshow message (Remove at least one ...)
                  window.find_element("noticec").hide_row()

                  #Update current standing and reset player move
                  in_standing = [in_standing[0] - player_move[0], in_standing[1] - player_move[1], in_standing[2] - player_move[2]]
                  player_move = [0,0,0]

                  #Check if game is in end state (one match or no matches remaining) and show menu items
                  if sum(in_standing) == 1:
                        window.find_element("winc").unhide_row()
                        showMenu1()
                        started = False
                  elif sum(in_standing) == 0:
                        window.find_element("lossc").unhide_row()
                        showMenu1()
                        started = False
                  else:
                        #If game is not in end state get computer move
                        performCompMove()

      #Check if player removes a match from row one
      if event.startswith("match1") and started:

            #Check if player has already removed matches from any other row
            if player_move[1] == 0 and player_move[2] == 0:
                  window.find_element(event).update(visible=False)         #Hide match
                  player_move[0] += 1                                                                       #Update player move
                  row1.remove(int(event[-1]))                                                    #Remove match from index list
      
      #Check if player removes a match from row two
      if event.startswith("match2") and started:
            
            #Check if player has already removed matches from any other row
            if player_move[0] == 0 and player_move[2] == 0:
                  window.find_element(event).update(visible=False)
                  player_move[1] += 1
                  row2.remove(int(event[-1]))
      
      #Check if player removes a match from row three
      if event.startswith("match3") and started:
            
            #Check if player has already removed matches from any other row
            if player_move[0] == 0 and player_move[1] == 0 :
                  window.find_element(event).update(visible=False)
                  player_move[2] += 1
                  row3.remove(int(event[-1]))


window.close()