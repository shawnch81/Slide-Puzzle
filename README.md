Slide-Puzzle
============

slide puzzle python code
# Slide Puzzle
# By Al Sweigart al@inventwithpython.com
# http://inventwithpython.com/pygame
# Released under a "Simplified BSD" license

import pygame, sys, random
from pygame.locals import *

# Create the constants (go ahead and experiment with different values)
行 = 4  # 數量 of columns in the 區塊
列 = 4 # 數量 of rows in the 區塊
拼圖尺寸 = 80
視窗寬 = 640
視窗高 = 480
FPS = 30
空格 = None

#                 R    G    B
黑 =         (  0,   0,   0)
白 =         (255, 255, 255)
淡藍 =    (  128,  255, 255)
紫 = (  128,  0,  128)
藍 =         (  0, 0,   255)

背景色 = 紫
拼圖色 = 藍
文字色 = 白
框色 = 淡藍
基本字體尺寸 = 20

按鈕色 = 白
按鈕文字顏色 = 黑
訊息顏色 = 白

X軸 = int((視窗寬 - (拼圖尺寸 * 行 + (行 - 1))) / 2)
Y軸 = int((視窗高 - (拼圖尺寸 * 列 + (列 - 1))) / 2)

上 = '上'
下 = '下'
左 = '左'
右 = '右'

def main():
    global FPS控制器, 顯示介面, 基本字型, 重置介面, 重置拼圖, 新介面, 新拼圖, 解答介面, 解答拼圖

    pygame.init()
    FPS控制器 = pygame.time.Clock()
    顯示介面 = pygame.display.set_mode((視窗寬, 視窗高))
    pygame.display.set_caption('Slide Puzzle')
    基本字型 = pygame.font.Font('freesansbold.ttf', 基本字體尺寸)

    # Store the option buttons and their rectangles in OPTIONS.
    重置介面, 重置拼圖 = 製造文字('Reset',    文字色, 拼圖色, 視窗寬 - 120, 視窗高 - 90)
    新介面,   新拼圖   = 製造文字('New Game', 文字色, 拼圖色, 視窗寬 - 120, 視窗高 - 60)
    解答介面, 解答拼圖 = 製造文字('Solve',    文字色, 拼圖色, 視窗寬 - 120, 視窗高 - 30)

    主要區塊, 解答順序 = 產生新拼圖(80)
    解答區塊 = 初始區塊() # a solved 區塊 is the same as the 區塊 in a start state.
    移動軌跡 = [] # list of 移動s made from the solved configuration

    while True: # main game loop
        滑向 = None # the 方向, if any, a 拼圖 should slide
        訊息 = 'Please click tile !' # contains the message to show in the 上per 左 corner.
        if 主要區塊 == 解答區塊:
            訊息 = 'Success!'

        繪圖區塊(主要區塊, 訊息)

        結束檢查()
        for event in pygame.event.get(): # event handling loop
            if event.type == MOUSEBUTTONUP:
                點X, 點y = 取得點擊座標(主要區塊, event.pos[0], event.pos[1])

                if (點X, 點y) == (None, None):
                    # check if the user clicked on an option button
                    if 重置拼圖.collidepoint(event.pos):
                        重設動畫(主要區塊, 移動軌跡) # clicked on Reset button
                        移動軌跡 = []
                    elif 新拼圖.collidepoint(event.pos):
                        主要區塊, 解答順序 = 產生新拼圖(80) # clicked on New Game button
                        移動軌跡 = []
                    elif 解答拼圖.collidepoint(event.pos):
                        重設動畫(主要區塊, 解答順序 + 移動軌跡) # clicked on Solve button
                        移動軌跡 = []
                else:
                    # check if the clicked 拼圖 was next to the 空格 spot

                    空格x, 空格y = 取得空格位置(主要區塊)
                    if 點X == 空格x + 1 and 點y == 空格y:
                        滑向 = 左
                    elif 點X == 空格x - 1 and 點y == 空格y:
                        滑向 = 右
                    elif 點X == 空格x and 點y == 空格y + 1:
                        滑向 = 上
                    elif 點X == 空格x and 點y == 空格y - 1:
                        滑向 = 下

            elif event.type == KEYUP:
                # check if the user pressed a key to slide a 拼圖
                if event.key in (K_左, K_a) and 有效移動(主要區塊, 左):
                    滑向 = 左
                elif event.key in (K_右, K_d) and 有效移動(主要區塊, 右):
                    滑向 = 右
                elif event.key in (K_上, K_w) and 有效移動(主要區塊, 上):
                    滑向 = 上
                elif event.key in (K_下, K_s) and 有效移動(主要區塊, 下):
                    滑向 = 下

        if 滑向:
            滑動動畫(主要區塊, 滑向, 'Click 拼圖 or press arrow keys to slide.', 8) # show slide on screen
            移動函式(主要區塊, 滑向)
            移動軌跡.append(滑向) # record the slide
        pygame.display.update()
        FPS控制器.tick(FPS)


def 終止():
    pygame.quit()
    sys.exit()


def 結束檢查():
    for event in pygame.event.get(QUIT): # get all the QUIT events
        終止() # 終止 if any QUIT events are present
    for event in pygame.event.get(KEYUP): # get all the KEYUP events
        if event.key == K_ESCAPE:
            終止() # 終止 if the KEYUP event was for the Esc key
        pygame.event.post(event) # put the other KEYUP event objects back


def 初始區塊():
    # Return a 區塊 data structure with 拼圖s in the solved state.
    # For example, if 行 and 列 are both 3, this function
    # returns [[1, 4, 7], [2, 5, 8], [3, 6, 空格]]
    計數器 = 1
    區塊 = []
    for x in range(行):
        column = []
        for y in range(列):
            column.append(計數器)
            計數器 += 行
        區塊.append(column)
        計數器 -= 行 * (列 - 1) + 行 - 1

    區塊[行-1][列-1] = 空格
    return 區塊


def 取得空格位置(區塊):
    # Return the x and y of 區塊 coordinates of the 空格 space.
    for x in range(行):
        for y in range(列):
            if 區塊[x][y] == 空格:
                return (x, y)


def 移動函式(區塊, 移動):
    # This function does not check if the 移動 is valid.
    空格x, 空格y = 取得空格位置(區塊)

    if 移動 == 上:
        區塊[空格x][空格y], 區塊[空格x][空格y + 1] = 區塊[空格x][空格y + 1], 區塊[空格x][空格y]
    elif 移動 == 下:
        區塊[空格x][空格y], 區塊[空格x][空格y - 1] = 區塊[空格x][空格y - 1], 區塊[空格x][空格y]
    elif 移動 == 左:
        區塊[空格x][空格y], 區塊[空格x + 1][空格y] = 區塊[空格x + 1][空格y], 區塊[空格x][空格y]
    elif 移動 == 右:
        區塊[空格x][空格y], 區塊[空格x - 1][空格y] = 區塊[空格x - 1][空格y], 區塊[空格x][空格y]


def 有效移動(區塊, 移動):
    空格x, 空格y = 取得空格位置(區塊)
    return (移動 == 上 and 空格y != len(區塊[0]) - 1) or \
           (移動 == 下 and 空格y != 0) or \
           (移動 == 左 and 空格x != len(區塊) - 1) or \
           (移動 == 右 and 空格x != 0)


def 取得隨機移動(區塊, 前次移動=None):
    # start with a full list of all four 移動s
    全部有效移動 = [上, 下, 左, 右]

    # remove 移動s from the list as they are disqualified
    if 前次移動 == 上 or not 有效移動(區塊, 下):
        全部有效移動.remove(下)
    if 前次移動 == 下 or not 有效移動(區塊, 上):
        全部有效移動.remove(上)
    if 前次移動 == 左 or not 有效移動(區塊, 右):
        全部有效移動.remove(右)
    if 前次移動 == 右 or not 有效移動(區塊, 左):
        全部有效移動.remove(左)

    # return a random 移動 from the list of remaining 移動s
    return random.choice(全部有效移動)


def 取得左上拼圖(拼圖X, 拼圖Y):
    左 = X軸 + (拼圖X * 拼圖尺寸) + (拼圖X - 1)
    上方 = Y軸 + (拼圖Y * 拼圖尺寸) + (拼圖Y - 1)
    return (左, 上方)


def 取得點擊座標(區塊, x, y):
    # from the x & y pixel coordinates, get the x & y 區塊 coordinates
    for 拼圖X in range(len(區塊)):
        for 拼圖Y in range(len(區塊[0])):
            左, 上方 = 取得左上拼圖(拼圖X, 拼圖Y)
            拼圖Rect = pygame.Rect(左, 上方, 拼圖尺寸, 拼圖尺寸)
            if 拼圖Rect.collidepoint(x, y):
                return (拼圖X, 拼圖Y)
    return (None, None)


def 畫拼圖(拼圖x, 拼圖y, 數量, adjx=0, adjy=0):
    # draw a 拼圖 at 區塊 coordinates 拼圖x and 拼圖y, optionally a few
    # pixels over (determined by adjx and adjy)
    左, 上方 = 取得左上拼圖(拼圖x, 拼圖y)
    pygame.draw.rect(顯示介面, 拼圖色, (左 + adjx, 上方 + adjy, 拼圖尺寸, 拼圖尺寸))
    文字介面 = 基本字型.render(str(數量), True, 文字色)
    文字方塊 = 文字介面.get_rect()
    文字方塊.center = 左 + int(拼圖尺寸 / 2) + adjx, 上方 + int(拼圖尺寸 / 2) + adjy
    顯示介面.blit(文字介面, 文字方塊)


def 製造文字(文字, 顏色, 背景色, 上方, 左):
    # create the Surface and Rect objects for some 文字.
    文字介面 = 基本字型.render(文字, True, 顏色, 背景色)
    文字方塊 = 文字介面.get_rect()
    文字方塊.topleft = (上方, 左)
    return (文字介面, 文字方塊)


def 繪圖區塊(區塊, message):
    顯示介面.fill(背景色)
    if message:
        文字介面, 文字方塊 = 製造文字(message, 訊息顏色, 背景色, 5, 5)
        顯示介面.blit(文字介面, 文字方塊)

    for 拼圖x in range(len(區塊)):
        for 拼圖y in range(len(區塊[0])):
            if 區塊[拼圖x][拼圖y]:
                畫拼圖(拼圖x, 拼圖y, 區塊[拼圖x][拼圖y])

    左, 上方 = 取得左上拼圖(0, 0)
    寬 = 行 * 拼圖尺寸
    高 = 列 * 拼圖尺寸
    pygame.draw.rect(顯示介面, 框色, (左 - 5, 上方 - 5, 寬 + 11, 高 + 11), 4)

    顯示介面.blit(重置介面, 重置拼圖)
    顯示介面.blit(新介面, 新拼圖)
    顯示介面.blit(解答介面, 解答拼圖)


def 滑動動畫(區塊, 方向, message, 動畫速度):
    # Note: This function does not check if the 移動 is valid.

    空格x, 空格y = 取得空格位置(區塊)
    if 方向 == 上:
        移動x = 空格x
        移動y = 空格y + 1
    elif 方向 == 下:
        移動x = 空格x
        移動y = 空格y - 1
    elif 方向 == 左:
        移動x = 空格x + 1
        移動y = 空格y
    elif 方向 == 右:
        移動x = 空格x - 1
        移動y = 空格y

    # prepare the base surface
    繪圖區塊(區塊, message)
    基礎畫面 = 顯示介面.copy()
    # draw a 空格 space over the moving 拼圖 on the 基礎畫面 Surface.
    移動左, 移動上方 = 取得左上拼圖(移動x, 移動y)
    pygame.draw.rect(基礎畫面, 背景色, (移動左, 移動上方, 拼圖尺寸, 拼圖尺寸))

    for i in range(0, 拼圖尺寸, 動畫速度):
        # animate the 拼圖 sliding over
        結束檢查()
        顯示介面.blit(基礎畫面, (0, 0))
        if 方向 == 上:
            畫拼圖(移動x, 移動y, 區塊[移動x][移動y], 0, -i)
        if 方向 == 下:
            畫拼圖(移動x, 移動y, 區塊[移動x][移動y], 0, i)
        if 方向 == 左:
            畫拼圖(移動x, 移動y, 區塊[移動x][移動y], -i, 0)
        if 方向 == 右:
            畫拼圖(移動x, 移動y, 區塊[移動x][移動y], i, 0)

        pygame.display.update()
        FPS控制器.tick(FPS)


def 產生新拼圖(滑動次數):
    # From a starting configuration, make 滑動次數 數量 of 移動s (and
    # animate these 移動s).
    順序 = []
    區塊 = 初始區塊()
    繪圖區塊(區塊, '')
    pygame.display.update()
    pygame.time.wait(500) # pause 500 milliseconds for effect
    前次移動 = None
    for i in range(滑動次數):
        移動 = 取得隨機移動(區塊, 前次移動)
        滑動動畫(區塊, 移動, 'Generate...', 動畫速度=int(拼圖尺寸 / 3))
        移動函式(區塊, 移動)
        順序.append(移動)
        前次移動 = 移動
    return (區塊, 順序)


def 重設動畫(區塊, 移動軌跡):
    # make all of the 移動s in 移動軌跡 in reverse.
    記錄移動軌跡 = 移動軌跡[:] # gets a copy of the list
    記錄移動軌跡.reverse()

    for 移動 in 記錄移動軌跡:
        if 移動 == 上:
            opposite移動 = 下
        elif 移動 == 下:
            opposite移動 = 上
        elif 移動 == 右:
            opposite移動 = 左
        elif 移動 == 左:
            opposite移動 = 右
        滑動動畫(區塊, opposite移動, '', 動畫速度=int(拼圖尺寸 / 2))
        移動函式(區塊, opposite移動)


if __name__ == '__main__':
    main()
