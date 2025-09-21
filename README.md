# scarymaze
-- Initialize game state
function _init()
    score = 0
    level = 1
    state = "menu" -- menu, playing, jumpscare, congrats, gameover
    jumpscare_type = nil -- "wall" or "completion"
    player = {x=64, y=64, dx=0, dy=0, speed=1.5, size=4}
    walls = {}
    next_level_block = nil
    message_timer = 0
    -- colors
    wall_color = 0
    playable_color = 12
    special_block_color = 8
end

-- Game loop
function _update()
    if state=="menu" then
        if btnp(4) or btnp(5) then
            state="playing"
            reset_level()
        end

    elseif state=="playing" then
        player.dx, player.dy = 0,0
        if btn(0) then player.dx=-1 end
        if btn(1) then player.dx=1 end
        if btn(2) then player.dy=-1 end
        if btn(3) then player.dy=1 end

        if player.dx~=0 and player.dy~=0 then
            player.x += player.dx*player.speed*0.707
            player.y += player.dy*player.speed*0.707
        else
            player.x += player.dx*player.speed
            player.y += player.dy*player.speed
        end

        -- wall collision
        for i,w in ipairs(walls) do
            if player.x-player.size < w.x+w.w and
               player.x+player.size > w.x and
               player.y-player.size < w.y+w.h and
               player.y+player.size > w.y then
                state="jumpscare"
                jumpscare_type="wall"
                message_timer=120
                sfx(1)
            end
        end

        -- next level block collision
        if next_level_block then
            local b = next_level_block
            if player.x-player.size < b.x+b.w and
               player.x+player.size > b.x and
               player.y-player.size < b.y+b.h and
               player.y+player.size > b.y then

               if level>=4 then
                    state="jumpscare"
                    jumpscare_type="completion"
                    message_timer=180
                    sfx(0)
               else
                    level += 1
                    score += 10
                    reset_level()
                    sfx(0)
               end
            end
        end

        if message_timer>0 then
            message_timer -=1
        end

    elseif state=="jumpscare" then
        message_timer -=1
        if message_timer <=0 then
            if jumpscare_type=="completion" then
                state="congrats"
            else
                state="playing"
                reset_level()
            end
        end

    elseif state=="congrats" then
        if btnp(4) or btnp(5) then
            _init()
        end

    elseif state=="gameover" then
        if btnp(4) or btnp(5) then
            _init()
        end
    end
end

-- Draw function
function _draw()
    cls()
    if state=="menu" then
        print_centered("Shrinking Walls",50,7)
        print_centered("Press Z or X to start",60,1)
        print_centered("Avoid the walls!",70,1)

    elseif state=="playing" then
        cls(playable_color)
        for i,w in ipairs(walls) do
            rectfill(w.x, w.y, w.x+w.w, w.y+w.h, w.color)
        end
        if next_level_block then
            local b = next_level_block
            rectfill(b.x, b.y, b.x+b.w, b.y+b.h, special_block_color)
        end
        circfill(player.x, player.y, player.size, 10)
        print("score:"..score,1,1,10)
        print("level:"..level,1,9,10)

    elseif state=="jumpscare" then
        if jumpscare_type=="wall" then
            -- full-screen flickering red X
            local flicker_color = (flr(message_timer/4)%2==0) and 8 or 7
            rectfill(0,0,127,127,flicker_color)
            -- draw clean thick X
            draw_fullscreen_x()
        else
            cls(0)
            print_centered("ユか🅾️웃 congrats! you finished!",50,11)
            draw_smiley(60,70,8)
        end

    elseif state=="congrats" then
        cls(7)
        print_centered("🎉 CONGRATULATIONS! 🎉",50,11)
        print_centered("Final Score: "..score,60,10)
        print_centered("Press Z or X to restart",70,1)
        draw_smiley(64,80,10)

    elseif state=="gameover" then
        print_centered("Game Over!",50,7)
        print_centered("Final Score: "..score,60,7)
        print_centered("Press Z or X to restart",70,1)
    end
end

-- Draw a clean full-screen X
function draw_fullscreen_x()
    local thickness = 6
    for i=0,thickness-1 do
        line(0+i,0,127-i,127,0)
        line(0+i,127,127-i,0,0)
    end
end

-- Reset level
function reset_level()
    walls = {}
    local block_size = 16
    if level==1 then
        add(walls,{x=0,y=0,w=128,h=16,color=wall_color})
        add(walls,{x=112,y=16,w=16,h=64,color=wall_color})
        add(walls,{x=48,y=64,w=64,h=16,color=wall_color})
        add(walls,{x=0,y=80,w=16,h=64,color=wall_color})
        add(walls,{x=0,y=128,w=128,h=16,color=wall_color})
        next_level_block={x=24,y=96,w=block_size,h=block_size,color=special_block_color}
        player.x,player.y=96,32

    elseif level==2 then
        add(walls,{x=0,y=0,w=128,h=32,color=wall_color})
        add(walls,{x=48,y=32,w=80,h=32,color=wall_color})
        add(walls,{x=0,y=80,w=80,h=32,color=wall_color})
        add(walls,{x=0,y=112,w=128,h=16,color=wall_color})
        next_level_block={x=112,y=64,w=block_size,h=block_size,color=special_block_color}
        player.x,player.y=24,48

    elseif level==3 then
        local corridor_width=16
        local mid_y=64
        local right_v_x=112
        add(walls,{x=0,y=0,w=right_v_x,h=mid_y-corridor_width/2,color=wall_color})
        add(walls,{x=0,y=mid_y+corridor_width/2,w=right_v_x,h=128-(mid_y+corridor_width/2),color=wall_color})
        next_level_block={x=right_v_x,y=128-block_size-4,w=block_size,h=block_size,color=special_block_color}
        player.x,player.y=16,mid_y

    else
        local hole_size = 16 - (level-1)*2
        local side = flr(rnd(4))
        local offset = flr(rnd(128-hole_size))
        if side==0 or side==2 then
            add(walls,{x=0,y=0,w=offset,h=128,color=wall_color})
            add(walls,{x=offset+hole_size,y=0,w=128-(offset+hole_size),h=128,color=wall_color})
            next_level_block={x=offset,y=64,w=hole_size,h=hole_size,color=special_block_color}
            player.x,player.y=offset+hole_size/2,64
        else
            add(walls,{x=0,y=0,w=128,h=offset,color=wall_color})
            add(walls,{x=0,y=offset+hole_size,w=128,h=128-(offset+hole_size),color=wall_color})
            next_level_block={x=64,y=offset,w=hole_size,h=hole_size,color=special_block_color}
            player.x,player.y=64,offset+hole_size/2
        end
    end
    message_timer=5
end

-- Draw smiley
function draw_smiley(cx,cy,c)
    circfill(cx,cy,6,c)
    pset(cx-2,cy-1,0)
    pset(cx+2,cy-1,0)
    line(cx-2,cy+2,cx+2,cy+2,0)
end

-- Helper: centered text
function print_centered(txt,y,c)
    local w=#txt*4
    print(txt,64-w/2,y,c)
end
