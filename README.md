import logic.Converter; import db.DatabaseManager;

import javax.swing.; import java.awt.; import java.awt.event.ActionEvent; import java.sql.SQLException;

public class MainFrame extends JFrame { private JComboBox<String> categoryBox, fromUnitBox, toUnitBox; private JTextField inputField; private JLabel resultLabel; private JButton convertBtn, historyBtn;

public MainFrame() {
    setTitle("Unit Converter");
    setSize(500, 300);
    setDefaultCloseOperation(EXIT_ON_CLOSE);
    setLayout(new GridLayout(6, 2));

    categoryBox = new JComboBox<>(new String[]{"Length", "Weight", "Temperature"});
    fromUnitBox = new JComboBox<>();
    toUnitBox = new JComboBox<>();
    inputField = new JTextField();
    resultLabel = new JLabel("Result: ");
    convertBtn = new JButton("Convert");
    historyBtn = new JButton("View History");

    categoryBox.addActionListener(e -> updateUnits());
    convertBtn.addActionListener(this::convert);
    historyBtn.addActionListener(e -> viewHistory());

    add(new JLabel("Category")); add(categoryBox);
    add(new JLabel("From Unit")); add(fromUnitBox);
    add(new JLabel("To Unit")); add(toUnitBox);
    add(new JLabel("Value")); add(inputField);
    add(convertBtn); add(historyBtn);
    add(resultLabel);

    updateUnits();
    setVisible(true);
}

private void updateUnits() {
    String category = (String) categoryBox.getSelectedItem();
    fromUnitBox.removeAllItems();
    toUnitBox.removeAllItems();
    for (String unit : Converter.getUnitsForCategory(category)) {
        fromUnitBox.addItem(unit);
        toUnitBox.addItem(unit);
    }
}

private void convert(ActionEvent e) {
    try {
        String category = (String) categoryBox.getSelectedItem();
        String from = (String) fromUnitBox.getSelectedItem();
        String to = (String) toUnitBox.getSelectedItem();
        double input = Double.parseDouble(inputField.getText());
        double result = Converter.convert(category, from, to, input);
        resultLabel.setText("Result: " + result);
        DatabaseManager.insertConversion(category, from, to, input, result);
    } catch (NumberFormatException ex) {
        JOptionPane.showMessageDialog(this, "Invalid input. Enter a number.");
    } catch (SQLException ex) {
        JOptionPane.showMessageDialog(this, "Database error: " + ex.getMessage());
    }
}

private void viewHistory() {
    try {
        String history = DatabaseManager.getHistory();
        JOptionPane.showMessageDialog(this, history);
    } catch (SQLException ex) {
        JOptionPane.showMessageDialog(this, "Database error: " + ex.getMessage());
    }
}

public static void main(String[] args) {
    new MainFrame();
}

}
