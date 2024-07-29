import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Stack;

public class ScientificCalculator extends JFrame implements ActionListener {
    private final JTextField display;

    public ScientificCalculator() {
        setTitle("Scientific Calculator");
        setSize(400, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        display = new JTextField();
        display.setFont(new Font("Arial", Font.BOLD, 24));
        display.setHorizontalAlignment(SwingConstants.RIGHT);
        display.setEditable(false);

        JPanel panel = new JPanel();
        panel.setLayout(new GridLayout(6, 4, 10, 10));

        String[] buttons = {
                "7", "8", "9", "/",
                "4", "5", "6", "*",
                "1", "2", "3", "-",
                "0", ".", "=", "+",
                "sin", "cos", "tan", "log",
                "(", ")", "C", "AC"
        };

        for (String text : buttons) {
            JButton button = new JButton(text);
            button.setFont(new Font("Arial", Font.BOLD, 18));
            button.addActionListener(this);
            panel.add(button);
        }

        add(display, BorderLayout.NORTH);
        add(panel);

        setVisible(true);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();
        String currentText = display.getText();

        switch (command) {
            case "=":
                display.setText(evaluateExpression(currentText));
                break;
            case "C":
                if (!currentText.isEmpty()) {
                    display.setText(currentText.substring(0, currentText.length() - 1));
                }
                break;
            case "AC":
                display.setText("");
                break;
            default:
                display.setText(currentText + command);
        }
    }

    private String evaluateExpression(String expression) {
        try {
            Stack<Double> values = new Stack<>();
            Stack<Character> operators = new Stack<>();
            for (int i = 0; i < expression.length(); i++) {
                char c = expression.charAt(i);
                if (Character.isDigit(c)) {
                    StringBuilder sb = new StringBuilder();
                    while (i < expression.length() && (Character.isDigit(expression.charAt(i)) || expression.charAt(i) == '.')) {
                        sb.append(expression.charAt(i++));
                    }
                    i--;
                    values.push(Double.parseDouble(sb.toString()));
                } else if (c == '(') {
                    operators.push(c);
                } else if (c == ')') {
                    while (operators.peek() != '(') {
                        values.push(applyOperator(operators.pop(), values.pop(), values.pop()));
                    }
                    operators.pop();
                } else if (isOperator(c)) {
                    while (!operators.isEmpty() && precedence(c) <= precedence(operators.peek())) {
                        values.push(applyOperator(operators.pop(), values.pop(), values.pop()));
                    }
                    operators.push(c);
                }
            }
            while (!operators.isEmpty()) {
                values.push(applyOperator(operators.pop(), values.pop(), values.pop()));
            }
            return String.valueOf(values.pop());
        } catch (Exception e) {
            return "Error";
        }
    }

    private boolean isOperator(char c) {
        return c == '+' || c == '-' || c == '*' || c == '/' || c == 's' || c == 'c' || c == 't' || c == 'l';
    }

    private int precedence(char c) {
        if (c == '+' || c == '-') {
            return 1;
        }
        if (c == '*' || c == '/') {
            return 2;
        }
        if (c == 's' || c == 'c' || c == 't' || c == 'l') {
            return 3;
        }
        return -1;
    }

    private double applyOperator(char operator, double b, double a) {
        switch (operator) {
            case '+':
                return a + b;
            case '-':
                return a - b;
            case '*':
                return a * b;
            case '/':
                if (b == 0) throw new UnsupportedOperationException("Cannot divide by zero");
                return a / b;
            case 's':
                return Math.sin(Math.toRadians(b));
            case 'c':
                return Math.cos(Math.toRadians(b));
            case 't':
                return Math.tan(Math.toRadians(b));
            case 'l':
                return Math.log10(b);
            default:
                throw new UnsupportedOperationException("Invalid operator");
        }
    }

    public static void main(String[] args) {
        new ScientificCalculator();
    }
}
