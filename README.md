import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, TouchableOpacity, FlatList, StyleSheet } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import Modal from 'react-native-modal';
import { FontAwesome } from '@expo/vector-icons';

export default function App() {
  const [tasks, setTasks] = useState([]);
  const [taskInput, setTaskInput] = useState('');
  const [isEditModalVisible, setEditModalVisible] = useState(false);
  const [editingTaskIndex, setEditingTaskIndex] = useState(null);
  const [editTaskInput, setEditTaskInput] = useState('');
  const [isCompletedModalVisible, setCompletedModalVisible] = useState(false);
  const [isMenuVisible, setMenuVisible] = useState(false);

  useEffect(() => {
    loadTasks();
  }, []);

  const loadTasks = async () => {
    try {
      const tasksData = await AsyncStorage.getItem('tasks');
      if (tasksData !== null) {
        setTasks(JSON.parse(tasksData));
      }
    } catch (error) {
      console.error(error);
    }
  };

  const saveTasks = async (newTasks) => {
    try {
      await AsyncStorage.setItem('tasks', JSON.stringify(newTasks));
      setTasks(newTasks);
    } catch (error) {
      console.error(error);
    }
  };

  const addOrUpdateTask = () => {
    if (taskInput.trim() === '') {
      alert('Please enter a task.');
      return;
    }

    if (editingTaskIndex !== null) {
      const updatedTasks = [...tasks];
      updatedTasks[editingTaskIndex].text = taskInput;
      saveTasks(updatedTasks);
      setEditingTaskIndex(null);
      setTaskInput('');
    } else {
      const newTask = { text: taskInput, completed: false };
      saveTasks([...tasks, newTask]);
      setTaskInput('');
    }
  };

  const deleteTask = (index) => {
    const updatedTasks = tasks.filter((_, i) => i !== index);
    saveTasks(updatedTasks);
  };

  const toggleTaskCompletion = (index) => {
    const updatedTasks = [...tasks];
    updatedTasks[index].completed = !updatedTasks[index].completed;
    saveTasks(updatedTasks);
  };

  const openEditModal = (index) => {
    setEditingTaskIndex(index);
    setEditTaskInput(tasks[index].text);
    setEditModalVisible(true);
  };

  const updateTask = () => {
    if (editTaskInput.trim() === '') {
      alert('Please enter a task.');
      return;
    }

    const updatedTasks = [...tasks];
    updatedTasks[editingTaskIndex].text = editTaskInput;
    saveTasks(updatedTasks);
    setEditModalVisible(false);
  };

  const openMenu = () => {
    setMenuVisible(true);
  };

  const handleMenuOption = (option) => {
    setMenuVisible(false);
    if (option === 'completed') {
      setCompletedModalVisible(true);
    }
  };

  const renderTaskItem = ({ item, index }) => (
    <View style={styles.taskItem}>
      <TouchableOpacity onPress={() => toggleTaskCompletion(index)}>
        <FontAwesome
          name={item.completed ? 'check-circle' : 'circle'}
          size={24}
          color={item.completed ? '#00796b' : 'white'}
        />
      </TouchableOpacity>
      <Text style={[styles.taskText, item.completed && styles.completedTask]}>
        {item.text}
      </Text>
      {!item.completed && (
        <View style={styles.actions}>
          <TouchableOpacity onPress={() => openEditModal(index)}>
            <FontAwesome name="edit" size={24} color="#0288d1" />
          </TouchableOpacity>
          <TouchableOpacity onPress={() => deleteTask(index)}>
            <FontAwesome name="trash" size={24} color="#d32f2f" />
          </TouchableOpacity>
        </View>
      )}
    </View>
  );
  

  return (
    <View style={styles.container}>
      <View style={styles.topBar}>
        <Text style={styles.title}>
          <FontAwesome name="tasks" size={36} color="#00796b" /> To-Do List
        </Text>
        <TouchableOpacity onPress={openMenu}>
          <FontAwesome name="ellipsis-v" size={24} color="black" />
        </TouchableOpacity>
      </View>

      <TextInput
        style={styles.input}
        placeholder="Add a new task..."
        value={taskInput}
        onChangeText={setTaskInput}
      />
      <TouchableOpacity style={styles.addButton} onPress={addOrUpdateTask}>
        <FontAwesome name="plus" size={24} color="white" />
        <Text style={styles.addButtonText}>Add Task</Text>
      </TouchableOpacity>

      <FlatList
        data={tasks.filter(task => !task.completed)}
        renderItem={renderTaskItem}
        keyExtractor={(item, index) => index.toString()}
      />

      {/* Edit Task Modal */}
      <Modal isVisible={isEditModalVisible}>
        <View style={styles.modalContent}>
          <Text style={styles.modalTitle}>Edit Task</Text>
          <TextInput
            style={styles.input}
            value={editTaskInput}
            onChangeText={setEditTaskInput}
          />
          <TouchableOpacity style={styles.addButton} onPress={updateTask}>
            <Text style={styles.addButtonText}>Update Task</Text>
          </TouchableOpacity>
        </View>
      </Modal>

      {/* Completed Tasks Modal */}
      <Modal isVisible={isCompletedModalVisible}>
        <View style={styles.modalContent}>
          <Text style={styles.modalTitle}>Completed Tasks</Text>
          <FlatList
            data={tasks.filter(task => task.completed)}
            renderItem={renderTaskItem}
            keyExtractor={(item, index) => index.toString()}
          />
          <TouchableOpacity onPress={() => setCompletedModalVisible(false)}>
            <Text style={styles.modalCloseText}>Close</Text>
          </TouchableOpacity>
        </View>
      </Modal>

      {/* Menu Modal */}
      <Modal isVisible={isMenuVisible} style={styles.menuModal}>
        <View style={styles.menuContent}>
          <TouchableOpacity onPress={() => handleMenuOption('completed')}>
            <Text style={styles.menuOption}>Completed Tasks</Text>
          </TouchableOpacity>
          <TouchableOpacity onPress={() => setMenuVisible(false)}>
            <Text style={styles.menuOption}>Cancel</Text>
          </TouchableOpacity>
        </View>
      </Modal>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 25,
    backgroundColor: '#f0f0f0',
  },
  topBar: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 30,
    marginTop: 20,
  },
  title: {
    fontSize: 36,
    fontWeight: '600',
    color: '#00796b',
    marginTop: 50,
  },
  input: {
    padding: 15,
    borderColor: '#00796b',
    borderWidth: 2,
    borderRadius: 25,
    marginBottom: 15,
    fontSize: 18,
  },
  addButton: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: '#00796b',
    padding: 15,
    borderRadius: 25,
    marginBottom: 20,
  },
  addButtonText: {
    color: 'white',
    fontWeight: '600',
    fontSize: 18,
    marginLeft: 10,
  },
  taskItem: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    padding: 15,
    backgroundColor: '#e0f2f1',
    borderRadius: 20,
    marginBottom: 15,
  },
  taskText: {
    fontSize: 18,
  },
  completedTask: {
    textDecorationLine: 'line-through',
    color: '#9e9e9e',
    paddingHorizontal: 75,
  },
  actions: {
    flexDirection: 'row',
    gap: 15,
  },
  modalContent: {
    backgroundColor: '#ffffff',
    padding: 20,
    borderRadius: 25,
    alignItems: 'center',
  },
  modalTitle: {
    fontSize: 24,
    marginBottom: 20,
  },
  modalCloseText: {
    marginTop: 20,
    color: '#00796b',
    fontSize: 18,
  },
  menuModal: {
    justifyContent: 'flex-end',
    margin: 0,
  },
  menuContent: {
    backgroundColor: '#ffffff',
    padding: 20,
    borderTopLeftRadius: 25,
    borderTopRightRadius: 25,
    alignItems: 'center',
  },
  menuOption: {
    fontSize: 18,
    paddingVertical: 10,
  },
});
