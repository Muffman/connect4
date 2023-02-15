# connect4
import pygame
import random

# dimensions of board
ROW_COUNT = 6
COL_COUNT = 7


PLAYER_PIECE = 1
AI_PIECE = 2

COLORS = {'yellow': (255, 255, 0), 'red': (255, 0, 0), 'blue': (
    0, 0, 255), 'black': (0, 0, 0), 'white': (255, 255, 255)}


# making the board as a 2D list/matrix with only zeroes
board = [[0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0]]

#return list of columns that are not full
def get_valid_locations():
    valid_locations = []
    for col in range(7):
        if board[0][col] == 0:
            valid_locations.append(col)
    return valid_locations

# function to make a move
# if the bottom row(5th row) already has a piece it places the piece 1 row above
def make_move(board, col, piece):   
    for row in range(5, -1, -1):
        if board[row][col] == 0:
            board[row][col] = piece
            move_log.append((row, col))
            break


def ai_move(board, piece):

    if len(move_log) < 2:
        col = 3
        return col

    else:
        r, c = move_log[-2]
        moves = [c-1, c, c+1]
        valid_locations = get_valid_locations()

        if piece == AI_PIECE:
            opp_piece = PLAYER_PIECE
        else:
            opp_piece = AI_PIECE
        
        # computer checking for its winning moves
        temp_board = board[:]
        
        for col in valid_locations:  
            make_move(temp_board, col, piece)
            if winning_move(temp_board, piece):
                undo_move(temp_board,move_log)
                return col
            else:
                undo_move(temp_board,move_log)
                continue
            
        # computer checking for player's winning move
        for col in valid_locations:  
            make_move(board, col, opp_piece)
            if winning_move(board, opp_piece):
                undo_move(temp_board,move_log)
                return col
            else:
                undo_move(temp_board,move_log)
                continue

        col = random.choice(moves)
        if col in valid_locations:
            return col
        else:
            col = random.choice(valid_locations)
            return col


def undo_move(board,log):
    r, c = log[-1]
    board[r][c] = 0
    log.pop(-1)


def pygame_board(board, window):  # drawing the board in pygame

    # drawing the blue rectangle as background
    pygame.draw.rect(
        window, (COLORS['blue']), (0, sq_size, COL_COUNT*sq_size, ROW_COUNT*sq_size), 0)

    for r in range(ROW_COUNT):
        for c in range(COL_COUNT):

            if board[r][c] == 0:
                # drawing a black circle where there is no piece
                pygame.draw.circle(
                    window, COLORS['black'], (c*sq_size+sq_size/2, r*sq_size+sq_size+2*sq_size/4), radius, 0)

            if board[r][c] == 1:
                # drawing a yellow circle where there is player 1's piece
                pygame.draw.circle(
                    window, (COLORS['yellow']), (c*sq_size+sq_size/2, r*sq_size+sq_size+2*sq_size/4), radius, 0)

            if board[r][c] == 2:
                # drawing a red circle where there is player 2's piece
                pygame.draw.circle(
                    window, (COLORS['red']), (c*sq_size+sq_size/2, r*sq_size+sq_size+2*sq_size/4), radius, 0)

    pygame.display.update()



def read_board(move_log, win):
    index = 0
    piece = PLAYER_PIECE
    temp_log = []

    viewing = True
    while viewing:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return
            
        keys = pygame.key.get_pressed()
        if keys[pygame.K_RIGHT]:
            if index < len(move_log):
                move = move_log[index]
                temp_log.append(move)
                r, c = move
                board[r][c] = piece
                pygame_board(board, win)
                pygame.time.delay(100)
                index += 1

                if piece == PLAYER_PIECE:
                    piece = AI_PIECE
                else:
                    piece = PLAYER_PIECE
            
        if keys[pygame.K_LEFT]:
            if index > 0:
                index -= 1
                undo_move(board,temp_log)
                pygame_board(board, win)
                pygame.time.delay(100)
                
                if piece == PLAYER_PIECE:
                    piece = AI_PIECE
                else:
                    piece = PLAYER_PIECE
                    
        if keys[pygame.K_ESCAPE]:
            viewing = False
            return
                

def tie_checker():
    tie_ctr = 0
    for r in range(ROW_COUNT):
        for c in range(COL_COUNT):
            if board[r][c] == 0:
                tie_ctr += 1

    if tie_ctr == 0:
        return True


def winning_move(board, piece):  # function to check if we have four alike pieces

    for c in range(COL_COUNT-3):  # checks for horizontal location for win
        for r in range(ROW_COUNT):
            if board[r][c] == piece and board[r][c+1] == piece and board[r][c+2] == piece and board[r][c+3] == piece:
                return True

    for c in range(COL_COUNT):  # checks for vertical location for win
        for r in range(ROW_COUNT-3):
            if board[r][c] == piece and board[r+1][c] == piece and board[r+2][c] == piece and board[r+3][c] == piece:
                return True

    for c in range(COL_COUNT-3):  # checks '+ve' slope diagonal location for win
        for r in range(ROW_COUNT-3):
            if board[r][c] == piece and board[r+1][c+1] == piece and board[r+2][c+2] == piece and board[r+3][c+3] == piece:
                return True

    for c in range(COL_COUNT-3):  # checks '-ve' slope diagonal location for win
        for r in range(3, ROW_COUNT):
            if board[r][c] == piece and board[r-1][c+1] == piece and board[r-2][c+2] == piece and board[r-3][c+3] == piece:
                return True

def draw_moving_piece(win, color, posx):
    pygame.draw.circle(
        win, (COLORS[color]), (posx, int(sq_size/2)), radius)

def pause(game_window, mode, game_paused, game_state, run, board, turn, move_log):
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LSHIFT]:
        pygame.display.set_caption("GAME PAUSED")
        game_paused = True
        game_window.blit(paused, (0,0))
    if keys[pygame.K_r]:
        pygame.display.set_caption(mode)
        game_paused = False
        pygame_board(board, game_window)
    if keys[pygame.K_p]:
        game_state = False
        run = False
    if keys[pygame.K_m]:
        board = [[0 for _ in range(7)]for _ in range(6)]
        pygame_board(board, game_window)
        #doesn't reset move_log. might affect analysis l8r
        move_log = []
        game_paused = False
        turn = True

    return game_paused, game_state, run, board, turn, move_log

def end_game(win, prompt, color, game_state, board):
    label = WIN_COLOR.render(
        prompt, 1, (COLORS[color]))
    board = [[0 for _ in range(7)]for _ in range(6)]
    win.blit(label, (40, 10))
    game_state = False
    pygame.display.update()
    pygame.time.wait(5000)
    
    return game_state, board



def multi_player(game_state, run, game_paused, clock, FPS, board, turn, move_log):
    game_window = pygame.display.set_mode((height, width))
    pygame.display.set_caption("MULTIPLAYER")
    pygame_board(board, game_window)
    # looping till game is over(game_state=False)
    while game_state:   

        clock.tick(FPS)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                game_state = False
                run = False

            if event.type == pygame.MOUSEMOTION:
                if not game_paused:
                    pygame.draw.rect(
                        game_window, COLORS['black'], (0, 0, width, sq_size))
                    posx, posy = pygame.mouse.get_pos()
                if turn and not game_paused:
                    draw_moving_piece(game_window, 'yellow', posx)
                elif not turn and not game_paused:
                    draw_moving_piece(game_window, 'red', posx)
                    
                pygame.display.update()

            if event.type == pygame.MOUSEBUTTONDOWN:
                if not game_paused:
                    pygame.draw.rect(
                        game_window, COLORS['black'], (0, 0, width, sq_size))

                valid_locations = get_valid_locations()
                if turn and not game_paused:  # player 1's turn

                    posx, posy = pygame.mouse.get_pos()
                    col = int(posx//sq_size)
                    if col in valid_locations:
                        make_move(board, col, PLAYER_PIECE)
                    else:
                        continue
                    pygame_board(board, game_window)
                    if winning_move(board, PLAYER_PIECE):
                        game_state, board = end_game(game_window, "Player 1 wins!!",'yellow',game_state, board)
                        
                    turn = not turn          # alternating b/w turn = True and False every move[]

                elif not turn and not game_paused:  # player 2's turn

                    posx, posy = pygame.mouse.get_pos()
                    col = int(posx//sq_size)
                    if col in valid_locations:
                        make_move(board, col, AI_PIECE)
                    else:
                        continue
                    pygame_board(board, game_window)
                    if winning_move(board, AI_PIECE):
                        game_state, board = end_game(game_window, "Player 2 wins!!",'red',game_state, board)

                    turn = not turn          # alternating b/w turn = True and False every move

                if tie_checker():
                    game_state, board = end_game(game_window, "Game is a Tie!",'red',game_state, board)

                    break

        game_paused, game_state, run, board, turn, move_log = pause(game_window,'MULTIPLAYER',game_paused,game_state,run,board,turn,move_log)



def single_player(game_state, run, game_paused, clock, FPS, board, turn, move_log):
    game_window = pygame.display.set_mode((width, height))
    pygame.display.set_caption("SINGLEPLAYER")
    pygame_board(board, game_window)
    # looping till game is over(game_state=False)
    while game_state:

        clock.tick(FPS)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                game_state = False
                run = False
            if event.type == pygame.MOUSEMOTION:
                if not game_paused:
                    pygame.draw.rect(
                        game_window, COLORS['black'], (0, 0, width, sq_size))
                    posx, posy = pygame.mouse.get_pos()
                if turn and not game_paused:
                    draw_moving_piece(game_window, 'yellow', posx)
                elif not turn and not game_paused:
                   draw_moving_piece(game_window, 'red', posx)
                pygame.display.update()

            if event.type == pygame.MOUSEBUTTONDOWN:
                valid_locations = get_valid_locations()
                if not game_paused:
                    pygame.draw.rect(
                        game_window, COLORS['black'], (0, 0, width, sq_size))

                if turn and not game_paused:  # player 1's turn

                    posx, posy = pygame.mouse.get_pos()
                    col = int(posx//sq_size)
                    if col in valid_locations:
                        make_move(board, col, PLAYER_PIECE)
                    else:
                        continue
                    pygame_board(board, game_window)
                    if winning_move(board, PLAYER_PIECE):
                        game_state, board = end_game(game_window,"Player wins!!",'yellow',game_state, board)

                    turn = not turn          # alternating b/w turn = True and False every move

                    if not turn and game_state and not game_paused:  # player 2's turn

                        pygame.time.wait(300)
                        col = ai_move(board, AI_PIECE)
                        make_move(board, col, AI_PIECE)
                        pygame_board(board, game_window)
                        
                        if winning_move(board, AI_PIECE):
                            game_state, board = end_game(game_window, "Computer wins!!",'red',game_state, board)

                    if tie_checker():
                        game_state, board = end_game(game_window, "Game is a Tie!!",'yellow',game_state, board)

                    turn = not turn          # alternating b/w turn = True and False every move
                    pygame.display.update()
                
            game_paused, game_state, run, board, turn, move_log = pause(game_window,'SINGLEPLAYER',game_paused,game_state,run,board,turn,move_log)

    pygame.display.update()



def multi_ai(game_state, run, game_paused, clock, FPS, board, turn, move_log):
    game_window = pygame.display.set_mode((height, width))
    pygame.display.set_caption("MULTI-AI")
    pygame_board(board, game_window)
    # looping till game is over(game_state=False)
    while game_state:   

        clock.tick(FPS)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                game_state = False
                run = False

        if not game_paused:
            pygame.draw.rect(
                game_window, COLORS['black'], (0, 0, width, sq_size))

        valid_locations = get_valid_locations()
        if turn and not game_paused:  # player 1's turn

            posx, posy = pygame.mouse.get_pos()
            col = ai_move(board, PLAYER_PIECE)
            if col in valid_locations:
                make_move(board, col, PLAYER_PIECE)
                pygame.time.delay(300)
            else:
                continue
            pygame_board(board, game_window)
            if winning_move(board, PLAYER_PIECE):
                game_state, board = end_game(game_window, "Computer 1 wins!!",'yellow',game_state, board)
                
            turn = not turn          # alternating b/w turn = True and False every move[]

        elif not turn and not game_paused:  # player 2's turn

            posx, posy = pygame.mouse.get_pos()
            col = ai_move(board, PLAYER_PIECE)
            if col in valid_locations:
                make_move(board, col, AI_PIECE)
                pygame.time.delay(300)
            else:
                continue
            pygame_board(board, game_window)
            if winning_move(board, AI_PIECE):
                game_state, board = end_game(game_window, "Computer 2 wins!!",'red',game_state, board)

            turn = not turn          # alternating b/w turn = True and False every move

        if tie_checker():
            game_state, board = end_game(game_window, "Game is a Tie!",'red',game_state, board)

            break

        game_paused, game_state, run, board, turn, move_log = pause(game_window,'MULTI-AI',game_paused,game_state,run,board,turn,move_log)



# Initializing Pygame

pygame.init()

# Creating a window

welcome_Width = 730
welcome_Height = 600
screen = pygame.display.set_mode((welcome_Width, welcome_Height))
pygame.display.set_caption("PLOT 4")

# Welcome Screen and Next Button image

Welcome = pygame.image.load("Welcome.png")

screen.blit(Welcome, (-25, 0))

pygame.display.update()

# Game Variables

game_Paused = False

sq_size = 100  # size of one box in the matrix

radius = sq_size/3+sq_size/8

width = COL_COUNT*sq_size  # dimensions of the game_window
height = (ROW_COUNT+1)*sq_size

WIN_COLOR = pygame.font.SysFont("comicsans", round(3*sq_size/4))

pygame.display.update()


# indicator if game is over or not, True:game is in progress , False:game is over
game_state = True
turn = True

move_log = []

game_paused = False


FPS = 60
clock = pygame.time.Clock()

paused = pygame.image.load('paused.png')
paused = pygame.transform.scale(paused, (width, height))

index = 0
piece = PLAYER_PIECE
temp_log = []


# game loop

run = True
while run:

    screen.fill((0, 0, 0))
    board = [[0 for _ in range(7)]for _ in range(6)]
    move_log = []

    # Event handler

    for event in pygame.event.get():
        if event.type == pygame.KEYDOWN:

            if event.key == pygame.K_n:
                instructions_win = pygame.display.set_mode((1100, 625))
                pygame.display.set_caption("HOW TO PLAY")
                Instructions = pygame.image.load("Instructions.png")
                instructions_win.blit(Instructions, (0, 0))
                pygame.display.update()

            elif event.key == pygame.K_p:
                mode_win = pygame.display.set_mode((1000, 625))
                pygame.display.set_caption("MODE OF GAME")
                Mode = pygame.image.load("Mode.png")
                mode_win.blit(Mode, (-30, 0))
                pygame.display.update()

            elif event.key == pygame.K_m:
                multi_player(game_state, run, game_paused, clock, FPS, board, turn, move_log)
                print(move_log)
                
            elif event.key == pygame.K_s:
                single_player(game_state, run, game_paused, clock, FPS, board, turn, move_log)
                print(move_log)

            elif event.key == pygame.K_u:
                multi_ai(game_state, run, game_paused, clock, FPS, board, turn, move_log)

            elif event.key == pygame.K_v:
                game_window = pygame.display.set_mode((width, height))
                pygame.display.set_caption("Analysis")
                pygame_board(board, game_window)
                
                move_log = [(5, 3), (4, 3), (3, 3), (5, 2), (5, 5), (4, 2), (5, 4), (5, 6), (3, 2), (4, 6), (4, 4), (2, 2), (4, 5), (2, 3), (3, 4), (2, 4), (5, 1), (3, 5), (3, 6), (1, 3)]
                read_board(move_log, game_window)
                pygame_board(board, game_window)
                pygame.display.update()

        if event.type == pygame.QUIT:
            run = False
            
    
print(move_log)
pygame.quit()

