import pygame
import sys
import random

pygame
pygame.init()

WIDTH, HEIGHT = 1100, 800
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("中央银行行长经济模拟游戏")

BACKGROUND = (25, 35, 50)
PANEL_BG = (40, 55, 75)
PANEL_BG2 = (45, 65, 85)
TEXT_COLOR = (220, 230, 255)
HIGHLIGHT = (70, 150, 255)
GOOD_COLOR = (100, 230, 150)
BAD_COLOR = (255, 100, 100)
NEUTRAL_COLOR = (255, 200, 100)
BUTTON_COLOR = (60, 80, 120)
BUTTON_HOVER = (80, 110, 160)
CHART_COLORS = [(255, 100, 100), (100, 230, 150), (100, 180, 255), (255, 180, 100)]

title_font = pygame.font.SysFont("microsoftyahei", 48, bold=True)
header_font = pygame.font.SysFont("microsoftyahei", 32, bold=True)
font = pygame.font.SysFont("microsoftyahei", 24)
small_font = pygame.font.SysFont("microsoftyahei", 20)


class Economy:
    def __init__(self):
        self.inflation = 2.5  
        self.unemployment = 5.0  
        self.gdp_growth = 2.0  
        self.interest_rate = 2.5  
        self.money_supply = 5.0  
        self.consumer_confidence = 70  
        self.stock_market = 10000  
        self.quarter = 1
        self.year = 2023
        self.history = {
            "inflation": [self.inflation],
            "unemployment": [self.unemployment],
            "gdp_growth": [self.gdp_growth],
            "interest_rate": [self.interest_rate],
            "money_supply": [self.money_supply],
            "stock_market": [self.stock_market]
        }

    def update(self):
        ir_effect = (self.interest_rate - 2.5) * 0.1
        ms_effect = (self.money_supply - 5.0) * 0.15

        self.inflation += ms_effect - ir_effect * 0.5 + random.uniform(-0.3, 0.3)
        self.inflation = max(0.1, min(self.inflation, 15.0))

        self.unemployment += ir_effect * 0.3 - ms_effect * 0.2 + random.uniform(-0.2, 0.2)
        self.unemployment = max(2.0, min(self.unemployment, 20.0))

        self.gdp_growth = 3.0 - ir_effect * 0.8 + ms_effect * 0.6 + random.uniform(-0.5, 0.5)
        self.gdp_growth = max(-5.0, min(self.gdp_growth, 10.0))

        self.consumer_confidence += (self.gdp_growth - 2.0) * 2 - (self.inflation - 2.5) * 3
        self.consumer_confidence = max(10, min(self.consumer_confidence, 100))

        market_change = (self.gdp_growth * 100) - (self.interest_rate * 50) + random.uniform(-100, 100)
        self.stock_market += market_change
        self.stock_market = max(5000, self.stock_market)

        self.quarter += 1
        if self.quarter > 4:
            self.quarter = 1
            self.year += 1

        self.history["inflation"].append(self.inflation)
        self.history["unemployment"].append(self.unemployment)
        self.history["gdp_growth"].append(self.gdp_growth)
        self.history["interest_rate"].append(self.interest_rate)
        self.history["money_supply"].append(self.money_supply)
        self.history["stock_market"].append(self.stock_market)

        for key in self.history:
            if len(self.history[key]) > 20:
                self.history[key] = self.history[key][-20:]


class Button:
    def __init__(self, x, y, width, height, text, action=None):
        self.rect = pygame.Rect(x, y, width, height)
        self.text = text
        self.action = action
        self.hovered = False

    def draw(self, surface):
        color = BUTTON_HOVER if self.hovered else BUTTON_COLOR
        pygame.draw.rect(surface, color, self.rect, border_radius=10)
        pygame.draw.rect(surface, HIGHLIGHT, self.rect, 2, border_radius=10)

        text_surf = font.render(self.text, True, TEXT_COLOR)
        text_rect = text_surf.get_rect(center=self.rect.center)
        surface.blit(text_surf, text_rect)

    def check_hover(self, pos):
        self.hovered = self.rect.collidepoint(pos)

    def handle_event(self, event):
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            if self.hovered and self.action:
                return self.action()
        return False


class Slider:
    def __init__(self, x, y, width, min_val, max_val, initial_val, label):
        self.rect = pygame.Rect(x, y, width, 30)
        self.min = min_val
        self.max = max_val
        self.value = initial_val
        self.label = label
        self.dragging = False
        self.knob_radius = 15
        self.knob_x = x + (initial_val - min_val) / (max_val - min_val) * width

    def draw(self, surface):
        pygame.draw.rect(surface, PANEL_BG2, self.rect, border_radius=5)
        pygame.draw.rect(surface, HIGHLIGHT, self.rect, 2, border_radius=5)

        pygame.draw.circle(surface, HIGHLIGHT, (int(self.knob_x), self.rect.centery), self.knob_radius)
        pygame.draw.circle(surface, TEXT_COLOR, (int(self.knob_x), self.rect.centery), self.knob_radius - 4)

        label_surf = small_font.render(f"{self.label}: {self.value:.2f}%", True, TEXT_COLOR)
        surface.blit(label_surf, (self.rect.x, self.rect.y - 25))

    def handle_event(self, event):
        if event.type == pygame.MOUSEBUTTONDOWN:
            if pygame.Rect(self.knob_x - self.knob_radius, self.rect.y - self.knob_radius,
                           self.knob_radius * 2, self.rect.height + self.knob_radius * 2).collidepoint(event.pos):
                self.dragging = True

        elif event.type == pygame.MOUSEBUTTONUP:
            self.dragging = False

        elif event.type == pygame.MOUSEMOTION and self.dragging:
            self.knob_x = max(self.rect.left, min(event.pos[0], self.rect.right))
            self.value = self.min + (self.knob_x - self.rect.left) / self.rect.width * (self.max - self.min)
            return True
        return False


def draw_chart(surface, x, y, width, height, data, title, color, min_val=None, max_val=None):
    pygame.draw.rect(surface, PANEL_BG, (x, y, width, height), border_radius=8)
    pygame.draw.rect(surface, HIGHLIGHT, (x, y, width, height), 2, border_radius=8)

    title_surf = small_font.render(title, True, TEXT_COLOR)
    surface.blit(title_surf, (x + 10, y + 10))

    if len(data) < 2:
        return

    data_min = min(data) if min_val is None else min_val
    data_max = max(data) if max_val is None else max_val
    if data_min == data_max:
        data_min -= 1
        data_max += 1

    pygame.draw.line(surface, HIGHLIGHT, (x + 40, y + height - 40), (x + width - 20, y + height - 40), 2)  # X轴
    pygame.draw.line(surface, HIGHLIGHT, (x + 40, y + height - 40), (x + 40, y + 40), 2)  # Y轴

    for i in range(5):
        y_pos = y + height - 40 - i * (height - 80) / 4
        value = data_min + i * (data_max - data_min) / 4
        value_surf = small_font.render(f"{value:.1f}", True, TEXT_COLOR)
        surface.blit(value_surf, (x + 15, y_pos - 10))
        pygame.draw.line(surface, (70, 90, 120), (x + 38, y_pos), (x + 42, y_pos), 1)

    points = []
    for i, val in enumerate(data):
        x_pos = x + 40 + i * (width - 60) / (len(data) - 1)
        y_pos = y + height - 40 - (val - data_min) / (data_max - data_min) * (height - 80)
        points.append((x_pos, y_pos))

    if len(points) > 1:
        pygame.draw.lines(surface, color, False, points, 3)
        for point in points:
            pygame.draw.circle(surface, color, point, 4)


def draw_metric_card(surface, x, y, width, height, title, value, unit, target_range=None):
    pygame.draw.rect(surface, PANEL_BG, (x, y, width, height), border_radius=10)
    pygame.draw.rect(surface, HIGHLIGHT, (x, y, width, height), 2, border_radius=10)

    title_surf = small_font.render(title, True, TEXT_COLOR)
    surface.blit(title_surf, (x + 15, y + 15))

    value_color = GOOD_COLOR
    if target_range:
        if value < target_range[0] or value > target_range[1]:
            value_color = BAD_COLOR

    value_surf = header_font.render(f"{value:.2f}{unit}", True, value_color)
    surface.blit(value_surf,
                 (x + width // 2 - value_surf.get_width() // 2, y + height // 2 - value_surf.get_height() // 2))

    if target_range:
        range_surf = small_font.render(f"目标: {target_range[0]}{unit} - {target_range[1]}{unit}", True, NEUTRAL_COLOR)
        surface.blit(range_surf, (x + width // 2 - range_surf.get_width() // 2, y + height - 35))


def draw_policy_panel(surface, x, y, width, height):
    pygame.draw.rect(surface, PANEL_BG2, (x, y, width, height), border_radius=15)
    pygame.draw.rect(surface, HIGHLIGHT, (x, y, width, height), 2, border_radius=15)
    title = header_font.render("经济政策调整", True, HIGHLIGHT)
    surface.blit(title, (x + width // 2 - title.get_width() // 2, y + 15))


def main():
    clock = pygame.time.Clock()
    economy = Economy()
    next_quarter_btn = Button(WIDTH - 220, HEIGHT - 80, 200, 60, "进入下一季度", lambda: True)

    interest_slider = Slider(50, HEIGHT - 150, 300, 0.0, 10.0, economy.interest_rate, "利率")
    money_supply_slider = Slider(50, HEIGHT - 80, 300, -5.0, 15.0, economy.money_supply, "货币供应量增长率")

    game_over = False
    quarters = 0
    max_quarters = 20
    score = 0

    inflation_target = (1.0, 3.0)
    unemployment_target = (4.0, 6.0)
    gdp_target = (1.5, 4.0)

    running = True
    while running:
        mouse_pos = pygame.mouse.get_pos()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

            if not game_over:
                if interest_slider.handle_event(event):
                    economy.interest_rate = interest_slider.value
                if money_supply_slider.handle_event(event):
                    economy.money_supply = money_supply_slider.value

                if next_quarter_btn.handle_event(event):
                    economy.update()
                    quarters += 1

                    in_inflation_target = inflation_target[0] <= economy.inflation <= inflation_target[1]
                    in_unemployment_target = unemployment_target[0] <= economy.unemployment <= unemployment_target[1]
                    in_gdp_target = gdp_target[0] <= economy.gdp_growth <= gdp_target[1]

                    if in_inflation_target and in_unemployment_target and in_gdp_target:
                        score += 10
                    elif in_inflation_target and in_unemployment_target:
                        score += 5
                    elif in_inflation_target or in_unemployment_target:
                        score += 2

                    if quarters >= max_quarters:
                        game_over = True
            else:
                if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
                    economy = Economy()
                    interest_slider.value = economy.interest_rate
                    money_supply_slider.value = economy.money_supply
                    interest_slider.knob_x = interest_slider.rect.left + (
                                economy.interest_rate - interest_slider.min) / (
                                                         interest_slider.max - interest_slider.min) * interest_slider.rect.width
                    money_supply_slider.knob_x = money_supply_slider.rect.left + (
                                economy.money_supply - money_supply_slider.min) / (
                                                             money_supply_slider.max - money_supply_slider.min) * money_supply_slider.rect.width
                    quarters = 0
                    score = 0
                    game_over = False

        next_quarter_btn.check_hover(mouse_pos)

        screen.fill(BACKGROUND)

        title = "中央银行行长经济模拟"
        title_surf = title_font.render(title, True, HIGHLIGHT)
        screen.blit(title_surf, (WIDTH // 2 - title_surf.get_width() // 2, 20))
        pygame.draw.line(screen, HIGHLIGHT, (WIDTH // 2 - 200, 80), (WIDTH // 2 + 200, 80), 2)

        time_text = f"时间: {economy.year}年 第{economy.quarter}季度"
        time_surf = header_font.render(time_text, True, NEUTRAL_COLOR)
        screen.blit(time_surf, (WIDTH // 2 - time_surf.get_width() // 2, 90))

        card_width = 220
        card_height = 100
        card_margin = 10
        card_y = 130

        draw_metric_card(screen, 50, card_y, card_width, card_height, "通货膨胀率", economy.inflation, "%",
                         inflation_target)
        draw_metric_card(screen, 50 + card_width + card_margin, card_y, card_width, card_height, "失业率",
                         economy.unemployment, "%", unemployment_target)
        draw_metric_card(screen, 50 + (card_width + card_margin) * 2, card_y, card_width, card_height, "GDP增长率",
                         economy.gdp_growth, "%", gdp_target)

        draw_metric_card(screen, 50, card_y + card_height + card_margin, card_width, card_height, "消费者信心",
                         economy.consumer_confidence, "", (60, 100))
        draw_metric_card(screen, 50 + card_width + card_margin, card_y + card_height + card_margin, card_width,
                         card_height, "股票市场", economy.stock_market, "")
        draw_metric_card(screen, 50 + (card_width + card_margin) * 2, card_y + card_height + card_margin, card_width,
                         card_height, "当前得分", score, "")

        remaining_quarters = max_quarters - quarters
        draw_metric_card(screen, WIDTH - 50 - card_width, card_y, card_width, card_height * 2 + card_margin, "剩余季度",
                         remaining_quarters, "")

        chart_width = (WIDTH - 100) // 2 - 10
        chart_height = 220
        chart_y = card_y + (card_height + card_margin) * 2 + 20

        draw_chart(screen, 50, chart_y, chart_width, chart_height,
                   economy.history["inflation"], "通货膨胀率历史", CHART_COLORS[0], 0, 10)

        draw_chart(screen, 60 + chart_width, chart_y, chart_width, chart_height,
                   economy.history["unemployment"], "失业率历史", CHART_COLORS[1], 0, 15)

        policy_panel_y = chart_y + chart_height + 20
        policy_panel_height = 180
        draw_policy_panel(screen, 50, policy_panel_y, WIDTH - 100, policy_panel_height)

        interest_slider.draw(screen)
        money_supply_slider.draw(screen)

        next_quarter_btn.draw(screen)

        policy_text1 = font.render(f"当前利率: {economy.interest_rate:.2f}%", True, TEXT_COLOR)
        screen.blit(policy_text1, (400, policy_panel_y + 50))

        policy_text2 = font.render(f"当前货币供应增长率: {economy.money_supply:.2f}%", True, TEXT_COLOR)
        screen.blit(policy_text2, (400, policy_panel_y + 90))

        if game_over:
            overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
            overlay.fill((0, 0, 0, 180))
            screen.blit(overlay, (0, 0))

            result_rect = pygame.Rect(WIDTH // 2 - 250, HEIGHT // 2 - 180, 500, 360)
            pygame.draw.rect(screen, PANEL_BG, result_rect, border_radius=15)
            pygame.draw.rect(screen, HIGHLIGHT, result_rect, 3, border_radius=15)

            game_over_text = title_font.render("游戏结束!", True, NEUTRAL_COLOR)
            screen.blit(game_over_text, (WIDTH // 2 - game_over_text.get_width() // 2, HEIGHT // 2 - 150))

            final_score = header_font.render(f"最终得分: {score}", True, TEXT_COLOR)
            screen.blit(final_score, (WIDTH // 2 - final_score.get_width() // 2, HEIGHT // 2 - 80))

            if score >= 150:
                rating = "卓越的中央银行行长!"
                color = GOOD_COLOR
            elif score >= 100:
                rating = "优秀的经济管理者!"
                color = NEUTRAL_COLOR
            elif score >= 60:
                rating = "合格的经济管理者"
                color = TEXT_COLOR
            else:
                rating = "需要更多经济学知识..."
                color = BAD_COLOR

            rating_text = header_font.render(rating, True, color)
            screen.blit(rating_text, (WIDTH // 2 - rating_text.get_width() // 2, HEIGHT // 2 - 20))

            final_inflation = small_font.render(f"最终通货膨胀率: {economy.inflation:.2f}%", True,
                                                GOOD_COLOR if inflation_target[0] <= economy.inflation <=
                                                              inflation_target[1] else BAD_COLOR)
            screen.blit(final_inflation, (WIDTH // 2 - 200, HEIGHT // 2 + 40))

            final_unemployment = small_font.render(f"最终失业率: {economy.unemployment:.2f}%", True,
                                                   GOOD_COLOR if unemployment_target[0] <= economy.unemployment <=
                                                                 unemployment_target[1] else BAD_COLOR)
            screen.blit(final_unemployment, (WIDTH // 2 - 200, HEIGHT // 2 + 70))

            final_gdp = small_font.render(f"最终GDP增长率: {economy.gdp_growth:.2f}%", True,
                                          GOOD_COLOR if gdp_target[0] <= economy.gdp_growth <= gdp_target[
                                              1] else BAD_COLOR)
            screen.blit(final_gdp, (WIDTH // 2 - 200, HEIGHT // 2 + 100))

            restart_text = font.render("按 R 键重新开始游戏", True, HIGHLIGHT)
            screen.blit(restart_text, (WIDTH // 2 - restart_text.get_width() // 2, HEIGHT // 2 + 150))

        pygame.display.flip()
        clock.tick(60)

    pygame.quit()
    sys.exit()


if __name__ == "__main__":
    main()
