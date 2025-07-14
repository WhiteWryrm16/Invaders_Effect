Cosmic Defender SDL3: A 1:1 C++26 Port of Cosmic_Defenders.py

This project rewrites the Python Turtle game into modern C++26 using SDL 3, SDL_ttf, the STL and CMake. You’ll get:

- src/main.cpp with game logic, shapes, timing, input  

🕹️ src/main.cpp

`cpp

include <SDL3/SDL.h>

include <SDL3/SDL_ttf.h>

include <vector>

include <random>

include <string>

include <cmath>

// Window
constexpr int WIN_W = 600;
constexpr int WIN_H = 600;

// Speeds (pixels/sec)
constexpr float PLAYER_SPEED  = 300.f;
constexpr float BULLET_SPEED  = 400.f;
constexpr float INVADER_SPEED = 100.f;
constexpr float POWERUPSPEED = INVADERSPEED + 50.f;

// Game state
int score = 0;
int alienLife = 100;
bool bulletActive = false;

// Random
std::mt19937 rng{std::random_device{}()};
std::uniformrealdistribution<float> distX(0 + 20.f, WIN_W - 20.f);
std::uniformrealdistribution<float> distY(100.f, 250.f);

// Utility: draw filled circle
void drawCircle(SDL_Renderer* ren, float cx, float cy, float r){
    for(int dy = -r; dy <= r; ++dy){
        int span = (int)std::sqrt(rr - dydy);
        SDL_RenderDrawLineF(ren, cx - span, cy + dy, cx + span, cy + dy);
    }
}

// Utility: draw filled triangle (player)
void drawTriangle(SDL_Renderer* ren, float cx, float cy, float size){
    SDL_Vertex verts[3];
    SDL_Color c{255,255,255,255};
    verts[0].position = {cx,            cy - size};
    verts[1].position = {cx - size,     cy + size};
    verts[2].position = {cx + size,     cy + size};
    for(auto &v: verts) { v.color = c; v.tex_coord = {0,0}; }
    Uint16 idx[] = {0,1,2};
    SDL_RenderGeometry(ren, nullptr, verts, 3, idx, 3);
}

struct Entity { float x,y; };
struct RectEntity { float x,y,w,h; };

// Check AABB collision
bool collide(const RectEntity&a,const RectEntity&b){
    return !(a.x+a.w < b.x || b.x+b.w < a.x ||
             a.y+a.h < b.y || b.y+b.h < a.y);
}

int main() {
  SDLInit(SDLINIT_VIDEO);
  TTF_Init();

  SDLWindow*   win = SDLCreateWindow("Cosmic Defender",
                        SDLWINDOWPOSCENTERED, SDLWINDOWPOSCENTERED,
                        WINW, WINH, 0);
  SDLRenderer* ren = SDLCreateRenderer(win, -1, SDLRENDERERACCELERATED);

  // Load font
  TTFFont* font = TTFOpenFont("arial.ttf", 18);
  if(!font) return 1;

  // Text color
  SDL_Color txtColor{255,255,255,255};

  // Player
  Entity player{WINW/2.f, WINH - 50.f};

  // Bullet
  RectEntity bullet{0,0,5,20};

  // Invaders
  std::vector<Entity> invaders;
  for(int i=0;i<7;i++)
    invaders.push_back({distX(rng), distY(rng)});

  // Aliena
  RectEntity aliena{WIN_W/2.f -20, 80, 40, 40};
  int alienDir = +1;

  // Powerups
  std::vector<RectEntity> powerups;
  Uint32 lastPowerup = SDL_GetTicks();

  Uint64 lastTime = SDL_GetPerformanceCounter();
  bool running = true;
  while(running){
    // Delta time
    Uint64 now = SDL_GetPerformanceCounter();
    float dt = (now - lastTime) / float(SDL_GetPerformanceFrequency());
    lastTime = now;

    // Event
    SDL_Event e;
    while(SDL_PollEvent(&e)){
      if(e.type == SDLEVENTQUIT) running = false;
      if(e.type == SDLEVENTKEYDOWN && e.key.keysym.sym == SDLKSPACE && !bulletActive){
        bulletActive = true;
        bullet.x = player.x - bullet.w/2;
        bullet.y = player.y - 20;
      }
    }

    // Input state
    const Uint8* ks = SDL_GetKeyboardState(NULL);
    if(ks[SDLSCANCODELEFT])  player.x -= PLAYER_SPEED * dt;
    if(ks[SDLSCANCODERIGHT]) player.x += PLAYER_SPEED * dt;
    player.x = std::clamp(player.x, 20.f, WIN_W - 20.f);

    // Move bullet
    if(bulletActive){
      bullet.y -= BULLET_SPEED * dt;
      if(bullet.y + bullet.h < 0) bulletActive = false;
    }

    // Move invaders & check bullet collisions
    for(auto &inv : invaders){
      inv.y += INVADER_SPEED * dt;
      if(bulletActive){
        RectEntity br{bullet.x,bullet.y,bullet.w,bullet.h};
        RectEntity ir{inv.x-15,inv.y-15,30,30};
        if(collide(br,ir)){
          score += 10;
          bulletActive = false;
          inv.x = distX(rng);
          inv.y = distY(rng);
        }
      }
    }

    // Move aliena
    aliena.x += alienDir  INVADER_SPEED  dt;
    if(aliena.x < 0 || aliena.x + aliena.w > WIN_W){
      alienDir *= -1;
      aliena.y += 40;
      alienLife -= 10;
    }

    // Spawn powerup
    if(SDL_GetTicks() - lastPowerup > 5000){
      powerups.push_back({distX(rng)-10, distY(rng)-10, 20, 20});
      lastPowerup = SDL_GetTicks();
    }

    // Move powerups & pickup
    for(auto it = powerups.begin(); it != powerups.end();){
      it->y += POWERUP_SPEED * dt;
      RectEntity pe = *it;
      RectEntity pl{player.x-20, player.y-20, 40, 40};
      if(collide(pe, pl)){
        score += 50;
        it = powerups.erase(it);
      }
      else if(pe.y > WIN_H) it = powerups.erase(it);
      else ++it;
    }

    // Render
    SDL_SetRenderDrawColor(ren,0,0,0,255);
    SDL_RenderClear(ren);

    // Player
    SDL_SetRenderDrawColor(ren,255,255,255,255);
    drawTriangle(ren, player.x, player.y, 20);

    // Bullet
    if(bulletActive){
      SDL_Rect br{int(bullet.x),int(bullet.y),int(bullet.w),int(bullet.h)};
      SDL_SetRenderDrawColor(ren,255,255,0,255);
      SDL_RenderFillRect(ren, &br);
    }

    // Invaders
    SDL_SetRenderDrawColor(ren,255,0,0,255);
    for(auto &inv: invaders) drawCircle(ren, inv.x, inv.y, 15);

    // Aliena
    SDL_SetRenderDrawColor(ren,0,255,0,255);
    SDL_Rect ar{int(aliena.x),int(aliena.y),int(aliena.w),int(aliena.h)};
    SDL_RenderFillRect(ren, &ar);

    // Powerups
    SDL_SetRenderDrawColor(ren,0,0,255,255);
    for(auto &pu: powerups){
      SDL_Rect pr{int(pu.x),int(pu.y),int(pu.w),int(pu.h)};
      SDL_RenderFillRect(ren, &pr);
    }

    // Text: Score & Life
    SDLSurface* s1 = TTFRenderTextBlended(font, ("SCORE: " + std::tostring(score)).c_str(), txtColor);
    SDLTexture* t1 = SDLCreateTextureFromSurface(ren, s1);
    SDL_Rect r1{10,10,s1->w,s1->h};
    SDL_RenderCopy(ren, t1, nullptr, &r1);
    SDLDestroyTexture(t1); SDLFreeSurface(s1);

    SDLSurface* s2 = TTFRenderTextBlended(font, ("ALIEN LIFE: " + std::tostring(alienLife)).c_str(), txtColor);
    SDLTexture* t2 = SDLCreateTextureFromSurface(ren, s2);
    SDLRect r2{WINW-160,10,s2->w,s2->h};
    SDL_RenderCopy(ren, t2, nullptr, &r2);
    SDLDestroyTexture(t2); SDLFreeSurface(s2);

    SDL_RenderPresent(ren);
  }

  // Cleanup
  TTF_CloseFont(font);
  SDL_DestroyRenderer(ren);
  SDL_DestroyWindow(win);
  TTF_Quit();
  SDL_Quit();
  return 0;
}
`
You’ve now got a faithful, frame-rate-independent port of nFireInvaders.py into modern C++26 and SDL 3. Dive in, tweak speeds, add features, and level up your C++ game dev skills! 🚀🛸
