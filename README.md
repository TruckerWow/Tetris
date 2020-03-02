# Tetris


![Tetris](https://github.com/TruckerWow/Tetris/blob/master/Tetris.gif)

<img src="https://github.com/TruckerWow/Tetris/blob/master/Tetris.gif" width = 300>

pic from http://www.kosbie.net/cmu/fall-19/15-112/notes/notes-tetris/2_7_RemovingFullRows.html







```
#################################################

import cs112_f19_week7_linter
import math, copy, random

from cmu_112_graphics import *
from tkinter import *
import random


#################################################

def playTetris():
    rows, cols, cellSize, margin = gameDimensions()
    width = cols*cellSize+2*margin
    height = rows*cellSize+2*margin
    return width, height

def gameDimensions():
    rows = 15
    cols = 10
    cellSize = 20
    margin = 25
    return rows, cols, cellSize, margin

def appStarted(app):
    app.rows, app.cols, app.cellSize, app.margin = gameDimensions()
    app.emptyColor = 'blue'
    app.board = [([app.emptyColor]*app.cols) for row in range(app.rows)]
    #app.board[0][0] = "red" # top-left is red
    #app.board[0][app.cols-1] = "white" # top-right is white
    #app.board[app.rows-1][0] = "green" # bottom-left is green
    #app.board[app.rows-1][app.cols-1] = "gray" # bottom-right is gray
        # Seven "standard" pieces (tetrominoes)
    app.isGameOver = False
    app.pause = False
    app.timerDelay = 500
    app.score = 0
    app.iPiece = [
        [  True,  True,  True,  True ]
    ]
    app.jPiece = [
        [  True, False, False ],
        [  True,  True,  True ]
    ]
    app.lPiece = [
        [ False, False,  True ],
        [  True,  True,  True ]
    ]
    app.oPiece = [
        [  True,  True ],
        [  True,  True ]
    ]
    app.sPiece = [
        [ False,  True,  True ],
        [  True,  True, False ]
    ]
    app.tPiece = [
        [ False,  True, False ],
        [  True,  True,  True ]
    ]
    app.zPiece = [
        [  True,  True, False ],
        [ False,  True,  True ]
    ]
    app.tetrisPieces = [ app.iPiece, app.jPiece, app.lPiece, app.oPiece,
                         app.sPiece, app.tPiece, app.zPiece ]
    app.tetrisPieceColors = [ "red", "yellow", "magenta",
                              "pink", "cyan", "green", "orange" ]
    newFallingPiece(app)

def moveFallingPiece(app,drow,dcol):
    app.fallingPieceRow += drow
    app.fallingPieceCol += dcol
    if fallingPieceIsLegal(app) == False:
        app.fallingPieceRow -= drow
        app.fallingPieceCol -= dcol
        return False
    return True

def placeFallingPiece(app):
    rows = app.fallingPieceRowSize
    cols = app.fallingPieceColSize
    for row in range(rows):
        for col in range(cols):
            #print(row,col,app.fallingPiece[row][col])
            if app.fallingPiece[row][col] == True:
                app.board[app.fallingPieceRow+row]\
                    [app.fallingPieceCol+col] = app.fallingPieceColor
    removeFullRows(app)
    
def removeFullRows(app):
    #newboard = [([app.emptyColor]*app.cols) for row in range(app.rows)]
    newboard = []
    fullRows = 0
    for row in range(app.rows):
        count = 0
        for col in range(app.cols):
            if app.board[row][col] != 'blue':
                count += 1
        if count == app.cols:
            fullRows += 1
        else:
            newboard.append(app.board[row])
    for i in range(fullRows):
        newboard.insert(0,[app.emptyColor]*app.cols)
    app.board = newboard
    app.score += fullRows**2

def timerFired(app):
    if app.timerDelay >200:
        app.timerDelay -= 5
    if app.isGameOver or app.pause: 
        return
    if not moveFallingPiece(app, +1, 0):
        placeFallingPiece(app)
        newFallingPiece(app)
        if not fallingPieceIsLegal(app):
            app.isGameOver = True

def fallingPieceIsLegal(app):
    rows = app.fallingPieceRowSize
    cols = app.fallingPieceColSize
    for row in range(rows):
        for col in range(cols):
            eachrow = app.fallingPieceRow+row
            eachcol = app.fallingPieceCol+col
            (x0,y0,x1,y1) = getCellBounds(app,eachrow,eachcol)
            if app.fallingPiece[row][col] == True:
                if not((x0 >= app.margin) and\
                    (x1 <= playTetris()[0]-app.margin) and\
                    (y0 >= app.margin) and\
                    (y1 <= playTetris()[1]-app.margin) and\
                    (app.board[eachrow][eachcol] == 'blue')):
                    return False
    return True

def hardDrop(app):
    #print(app.rows,app.fallingPieceRow)
    distance = app.rows-app.fallingPieceRow-app.fallingPieceRowSize
    moveFallingPiece(app,distance,0)
    for row in range(app.rows):
        for col in range(app.fallingPieceCol,\
                        app.fallingPieceCol+app.fallingPieceColSize):
            if app.board[row][col] != 'blue':
                return(row-app.fallingPieceRow-app.fallingPieceRowSize)
    return(distance)


def keyPressed(app,event):
    if (event.key == 'r'):       appStarted(app)
    elif app.isGameOver: return
    elif   (event.key == 'Up'):  rotateFallingPiece(app)
    elif (event.key == 'Down'):  moveFallingPiece(app,+1,0)
    elif (event.key == 'Left'):  moveFallingPiece(app,0,-1)
    elif (event.key == 'Right'): moveFallingPiece(app,0,+1)
    elif (event.key == 'Space'): moveFallingPiece(app,hardDrop(app),0)
    elif (event.key == 'Enter'): app.pause = not app.pause

def newFallingPiece(app):
    randomIndex = random.randint(0, len(app.tetrisPieces) - 1)
    app.fallingPiece = app.tetrisPieces[randomIndex]
    app.fallingPieceColor = app.tetrisPieceColors[randomIndex]
    app.fallingPieceRow = 0
    app.fallingPieceRowSize = len(app.fallingPiece)
    app.fallingPieceColSize = len(app.fallingPiece[0])
    app.fallingPieceCol = int(app.cols//2-app.fallingPieceColSize//2)

def rotateFallingPiece(app):
    tempPieceRow = app.fallingPieceRow
    tempPieceCol = app.fallingPieceCol
    tempPieceRowSize = app.fallingPieceRowSize
    tempPieceColSize = app.fallingPieceColSize
    tempPiece = app.fallingPiece
    newPiece = [([None]*tempPieceRowSize) for row in range(tempPieceColSize)]
    for row in range(tempPieceRowSize):
        for col in range(tempPieceColSize):
            newPiece[tempPieceColSize-1-col][row] = tempPiece[row][col]
    app.fallingPiece = newPiece
    app.fallingPieceRowSize = len(app.fallingPiece)
    app.fallingPieceColSize = len(app.fallingPiece[0])
    #newRow = oldRow + oldNumRows//2 - newNumRows//2
    app.fallingPieceRow = tempPieceRow+tempPieceRowSize//2\
                          -app.fallingPieceRowSize//2
    app.fallingPieceCol = tempPieceCol+tempPieceColSize//2\
                          -app.fallingPieceColSize//2
    if fallingPieceIsLegal(app) == False:
        app.fallingPieceRow = tempPieceRow
        app.fallingPieceCol = tempPieceCol
        app.fallingPiece = tempPiece
        app.fallingPieceRowSize = tempPieceRowSize
        app.fallingPieceColSize = tempPieceColSize
    


def drawFallingPiece(app,canvas):
    rows = app.fallingPieceRowSize
    cols = app.fallingPieceColSize
    for row in range(rows):
        for col in range(cols):
            #print(row,col,app.fallingPiece[row][col])
            if app.fallingPiece[row][col] == True:
                fill = app.fallingPieceColor
                drawCell(app, canvas, app.fallingPieceRow+row, 
                        app.fallingPieceCol+col, fill)

def getCellBounds(app, row, col):
    gridWidth  = app.width - 2*app.margin
    gridHeight = app.height - 2*app.margin
    x0 = app.margin + gridWidth * col / app.cols
    x1 = app.margin + gridWidth * (col+1) / app.cols
    y0 = app.margin + gridHeight * row / app.rows
    y1 = app.margin + gridHeight * (row+1) / app.rows
    return (x0, y0, x1, y1)

def drawBoard(app, canvas):
    #canvas.create_rectangle(0, 0, app.width, app.height, fill='orange')
    for row in range(app.rows):
        for col in range(app.cols):
            fill = app.board[row][col]
            drawCell(app, canvas, row, col, fill)

def drawCell(app, canvas, row, col, fill):
    (x0, y0, x1, y1) = getCellBounds(app, row, col)
    canvas.create_rectangle(x0, y0, x1, y1, fill=fill,width=4)
    
def redrawAll(app, canvas):
    #drawCell(0, 0, app.width, app.height, fill='orange')
    canvas.create_rectangle(0, 0, app.width, app.height, fill='orange')
    drawBoard(app, canvas)
    drawFallingPiece(app,canvas)
    canvas.create_text(app.width/2,app.margin/2,
        text=f'score = {app.score}',font='Arial 10 bold',fill='blue')
    canvas.create_text(app.width/2,app.height-app.margin/2,
        text='press Enter to pause',font='Arial 12 bold',fill='black')
    if app.pause == True:
        canvas.create_text(app.width/2,app.height/2,
        text='Game Paused',font='Arial 20 bold',fill='red')
    if app.isGameOver == True:
        canvas.create_rectangle(app.margin, app.margin+app.cellSize,
        app.width-app.margin, app.margin+3*app.cellSize, fill='black')
        canvas.create_text(app.width/2,app.margin+2*app.cellSize,
        text='You died!',font='Arial 20 bold',fill='yellow')
    

runApp(width=playTetris()[0], height=playTetris()[1])

#################################################
# main
#################################################

def main():
    cs112_f19_week7_linter.lint()
    playTetris()

if __name__ == '__main__':
    main()
```
